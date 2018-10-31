---
layout: post
category : article
title: "Reactive APIs and Clean Architecture... darn"
comments: true
tags : [technical]
---

Reactive programming seems to have taken off recently, who doesn't love non-blocking I/O, right? But, as you may come to learn after reading this, reactive programming doesn't come without compromise.

To me, using reactive frameworks like Spring Webflux or Project Reactor (which Webflux uses) is a mere technical detail. Or at least I would like it to be. From previous articles you may have sensed I have a preference towards using Clean Architecture principles in my software, and it does come with a couple of rules. One of which is that the domain and infrastructure layers are as technologically neutral as possible. 

However, taking the reactive route, you soon realize that it is an invasive path. The concepts of reactive streams tend to permeate throughout your application, be it by using RxJava's `Flowable` or `Single`, or Project Reactor's equivalent in `Flux` and `Mono` (instead of using collections and simple object types in non-reactive applications). If you want to make sure your application is fully reactive, you'll need to use these in your application and domain layers as well. Urgh. A new dependency there rises. But one might say that these APIs aren't technical in nature and to some degree I can relate to that opinion. 

So you end up with code like this in your domain gateway:

``` kotlin
interface OrderGateway {
    fun getOrders() : Flux<Order> // or Flowable<Order>
    fun getOrder(orderId: String) : Mono<Order> // or Single<Order>
}
```

and code like this in your application use cases:

``` kotlin
interface GetOrders {
    fun <T> getOrders(presenter: (GetOrdersResponse) -> T) : Flux<T> // or Flowable<T>
}
```

I can live with that. But. There's a big but. 

There's something called transactionality and that's where things become dicey. Let's take a standard reactive setup using Spring.

You have a reactive controller, returning a `Flux`. It calls a use case, which calls a domain gateway, both of which return `Flux` as well and use the `map` function to decouple the data structures between the layers. The gateway is implemented using the new `Stream` support in Spring Data JPA and switching between a Stream and a `Flux` is dead-easy (and off course, also uses a `map` to decouple the database from the domain).

Here comes the kicker. All those `map` functions only get triggered if there is a consumer (or in reactive terms, a subscriber) on the other side. In this case, this is a reactive controller, which consumes the data and outputs it to the outside world. The problem here is that my transactional boundary is defined on the use case layer. Thus: suddenly the Stream API no longer works, as my code already exited the confines of it's transactional boundary and Spring Data JPA has closed the connection (as it should have).

I've come at the point where I'm wondering whether it is possible to make a reactive API where the transaction boundaries are somewhere in the middle of the call stack. One fix I've found is to do a `collectList().block()` call in the use case so that the `map` function actually already gets called in the use case (where it still has a transactional and therefor an open connection). But it totally defeats the purpose of a reactive API... Another possible solution is to abandon the `Stream` support in Spring Data JPA and use the scheduled support of Project Reactor, ending up with code in my gateway like this:

``` kotlin
return Mono.fromCallable { orderJpaRepository.findAll().map { it.toDomain() } }
                .publishOn(scheduler)
                .flatMapIterable { it }
```

In any case, the resulting call stack won't be reactive, but as there are no reactive JDBC drivers and there is no native reactive support in JDBC, I think the last possibility might be the cleanest one, deferring the reactive jerry-rigging to the infrastructure layers. And it has the added feature of actually being non-blocking.

I know the issue I'm having is probably because I'm trying to merge a non-reactive concept like JPA (which relies on JDBC) with reactive frameworks like Webflux. But it would have been cool if it worked out of the box. One can offcourse wonder whether it's actually useful to create a reactive API using blocking technologies like JPA. 

If you think you know of a solution, have a look at the code [on Github](https://github.com/lievendoclo/clean-restbucks/tree/reactive) and see whether you can implement the `OrderGateway` using the Stream API methods provided in the `OrderJpaGateway`. 