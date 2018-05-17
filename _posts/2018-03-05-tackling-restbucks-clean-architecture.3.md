---
layout: post
category : article
title: "Tackling Restbucks with Clean Architecture, episode 3"
comments: true
tags : [technical]
---

[Part 1 here]({% post_url 2018-02-25-tackling-restbucks-clean-architecture %})
[Part 2 here]({% post_url 2018-03-04-tackling-restbucks-clean-architecture.2 %})

In the previous 2 episodes we built the application and domain layer, and built the infrastructure layer for exposing a couple of REST endpoints. Now we're going to the other end of our application and look at how we can persist the data in our system. In the domain layer, we're sending our events and we have a gateway to implement. For this example, I'll use JPA and Spring Data (using Hibernate as the implementation), but you're free to choose whatever technology you're the most comfortable with.

First of all, we need to define our JPA entities.

{% highlight kotlin %}
@Entity
data class OrderEntity(@Id val id: String,
                       val customerName: String,
                       @Enumerated var status: Status,
                       val cost: BigDecimal,
                       @OneToMany(cascade = [CascadeType.ALL], fetch = FetchType.EAGER)
                       @JoinColumn(name = "order_id")
                       val items: List<OrderItemEntity>)

@Entity
data class OrderItemEntity(@GeneratedValue(generator = "UUID")
                           @GenericGenerator(
                                   name = "UUID",
                           strategy = "org.hibernate.id.UUIDGenerator")
                           @Id val id: String?,
                           val product: String,
                           val quantity: Int,
                           @Enumerated val size: Size,
                           @Enumerated val milk: Milk)
{% endhighlight %}

(small tip: Don't call your entities Order, SQL really doesn't like that and you'll hate yourself every time you need to use backticks because you wanted to use a reserved SQL keyword as a table name)

With Spring Data JPA, building a basic CRUD repository is a breeze.

{% highlight kotlin %}
interface OrderJpaRepository : JpaRepository<OrderEntity, String> 
{% endhighlight %}

One line of code, so much power. Have I mentioned yet I love Spring Data (not Spring Data REST, no, the persistence part)?

And now we have all the basic components to build our event handlers and our `OrderGateway` implementation. We'll start with the latter.

{% highlight kotlin %}
@Component
class JpaOrderGateway(val orderJpaRepository: OrderJpaRepository) : OrderGateway {
    override fun getOrder(orderId: String): Order {
        return orderJpaRepository.getOne(orderId).toDomain()
    }

    override fun getOrders(): List<Order> {
        return orderJpaRepository.findAll().map { it.toDomain() }
    }

    fun OrderEntity.toDomain() : Order {
        return Order(id, customerName, status, items.map { it.toDomain() })
    }

    fun OrderItemEntity.toDomain() : OrderItem {
        return OrderItem(product, quantity, size, milk)
    }
}
{% endhighlight %}

Not much to it, really, you need to do some translation between the persistent entities and the domain model, but for the rest, it's very straightforward (and clean). Once again, Kotlin's extension functions really help to make the code a lot more readable. So now that we have the read part handling, now we'll tackle the write section by handling the events.

{% highlight kotlin %}
@Component
class OrderCreatedConsumer(val orderJpaRepository: OrderJpaRepository) : DomainEventConsumer<OrderCreated> {
    override fun consume(event: OrderCreated) {
        val orderEntity = event.order.toJpa()
        orderJpaRepository.save(orderEntity)
    }

    fun Order.toJpa() : OrderEntity {
        return OrderEntity(id, customer, status, cost, items.map { it.toJpa() })
    }

    fun OrderItem.toJpa() : OrderItemEntity {
        return OrderItemEntity(null, product, quantity, size, milk)
    }
}


@Component
class OrderDeletedConsumer(val orderJpaRepository: OrderJpaRepository) : DomainEventConsumer<OrderDeleted> {
    override fun consume(event: OrderDeleted) {
        orderJpaRepository.deleteById(event.id)
    }
}

@Component
class OrderDeliveredConsumer(val orderJpaRepository: OrderJpaRepository) : DomainEventConsumer<OrderDelivered> {
    override fun consume(event: OrderDelivered) {
        val order = orderJpaRepository.getOne(event.id)
        order.status = Status.DELIVERED
        orderJpaRepository.save(order)
    }
}

@Component
class OrderPaidConsumer(val orderJpaRepository: OrderJpaRepository) : DomainEventConsumer<OrderPaid> {
    override fun consume(event: OrderPaid) {
        val order = orderJpaRepository.getOne(event.id)
        order.status = Status.PAID
        orderJpaRepository.save(order)
    }
}
{% endhighlight %}

Now, when you're implementing the events, you sometimes feel that the events really end up in very similar implementations. For example, `OrderPaid` and `OrderDelivered` could be combined in `OrderStatusChanged` if you added a `Status` to the event. However, driving your event design through your implementation may not always be the best idea. For example, what if you wanted another consumer to pick up on `OrderDelivered` and do something particular for that event. If you combined the events, you'd have to add an `if` structure to handle such a case. But as with all things, this is open to interpretation and compromise and there's no black or white answer here.

And that's it. We've now implemented Restbucks in a clean and decoupled manner. 

As you may have noticed by now, implementing infrastructure layers really don't account for much of the work. You'll be spending way more time in your application and domain layers, as the value for the customer really lies within those. Don't get me wrong, you do need the infrastructure layers to make everything work, but they should be easily interchangeable. For example, do the exercise to swap out one of the infrastructure layers. If you've decoupled correctly, you shouldn't feel the need to change the domain or application layer in order to do so. If you do, you've either made a compromise that has accrued interest or some framework dependency was introduced into your core layers. 

In the end, I hope I've shown you that building an application in a way that's in line with what Clean Architecture is trying to communicate to each and everyone of us really isn't that much more work. It's an investment that will pay off in the future. Or to quote Robert C. Martin:

> The higher the quality, the faster you go. The only way to go fast is to go well.

In the last episode, I'll show you some cross-cutting corners like validation and transactionality, and how to integrate those without introducing technical framework dependencies in your core layers. 

[Part 4 here]({% post_url 2018-05-17-tackling-restbucks-clean-architecture.4 %})