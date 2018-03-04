---
layout: post
category : article
title: "Tackling Restbucks with Clean Architecture, episode 1"
comments: true
tags : [technical]
---

As the concept of clean architecture tends to be about how to design software, it's sometimes hard to see the entire picture and truly appreciate the benefits clean architecture can bring to the table. More than once I've gotten the comment that clean architecture looks fine on paper, but the added complexity is not something that can be sold easily to managers or architects. So perhaps it's time to roll up our sleeves and get coding. For those that have read 'REST in Practice', there is a nice example in the on how a REST service for a coffeeshop (no, not the Netherlands variant) might look like. It has a clear scope, so it's ideal for this session. The entire exercise will be too long to put into a single article, so this will be a article in multiple episode. Episode 1: the application API layer and the REST interface.

From a use case standpoint, we can identify 6 use cases:

* Create a new order for coffee
* Show all the orders that haven't been delivered yet
* Get the status of an order
* Pay for an order
* Delete an order before it is delivered to the customer 
* Deliver an order to the customer

Starting from a [Spring Boot project template](https://github.com/cleanarchitecturebe/spring-boot-kotlin-template), we can start coding the application API. After we're finished, we're left with 6 interfaces and request/response models. For example, the CreateOrder use case would look like this:

{% highlight kotlin %}
interface CreateOrder {
    fun <T> create(request: CreateOrderRequest, presenter: (CreateOrderResponse) -> T) : T
}

data class CreateOrderRequest(val customer: String, val items: List<CreateOrderRequestItem>)
data class CreateOrderRequestItem(val product: String, val quantity: Int, val size: Size, val milk: Milk)

data class CreateOrderResponse(val id: String, val customer: String, val amount: BigDecimal)
{% endhighlight %}

And the GetOrders use case would look like this:

{% highlight kotlin %}
interface GetOrders {
    fun <T> getOrders(presenter: (List<GetOrdersResponse>) -> T) : T
}

data class GetOrdersResponse(val id: String,
                             val customer: String,
                             val status: Status,
                             val items: List<GetOrdersResponseItem>)
data class GetOrdersResponseItem(val product: String,
                                 val quantity: Int,
                                 val size: Size,
                                 val milk: Milk)
{% endhighlight %}

If you look closely, there is something that should catch your eye: Milk, Size and Status. These are simple enums, for examle Milk:

{% highlight kotlin %}
enum class Milk {
    WHOLE,
    SKIMMED,
    SOY
}
{% endhighlight %}

We could opt to put these into the application API layer, but that would mean we'd have to copy these inside the domain layer (the application API layer and domain layer do not have a dependency, remember?). There might be valid reasons to do so, for example if the application layer has the possibility to change from the domain variant you should copy them, but in this case they are something that is called 'shared vocabulary'. This is the one module in clean architecture on which every other module depends on. This module should be cared for tremendously. In most cases, the only thing that goes into these are simple shared datastructures (like monetary values if you're not using `javax.money`), shared enums and perhaps a couple of exceptions. In this case, we'll put int the three enums `Milk`, `Size` and `Status`. 

Another thing that you notice is the use of higher-order functions as parameters. This way we can defer the presentation logic to the consumer, yet ensure that the use case delivers the data structure needed by the consumer. It's a nice trick that you can use in Kotlin.

Now that we have our business interactions modelled, we can go in 2 directions, either we start going towards the web infrastructure layer, of we go towards the persistence infrastructure layer. As the shortest route is towards the web layer, I'll be going that route now, we'll just have to mock the implementations for now.

From a REST standpoint, we have a couple of endpoints:

- `/order`: a `POST` for creating an order and a `GET` for getting the orders
- `/order/{id}`: a `DELETE` for the deletion of an order
- `/order/{id}/status`: a `GET` for the status
- `/order/{id}/delivery`: a `POST` to deliver the order
- `/order/{id}/payment`: a `POST` to pay for the order

There are a couple of variants that are possible here. You can opt to use a `PATCH` for updating the status after delivery or adding a payment. Or you can use a `PUT` to change the status. As with most RESTful APIs, there really isn't correct answer when it comes to actions that cannot directly (and reasonably) be translated to resources. These kind of discussions tend to become religious in nature and you're best to avoid them. Pick a style and stick to it.

In the end, we end up with the Spring MVC controller looking something like this:

{% highlight kotlin %}
@RequestMapping("/order")
@RestController
class OrderResource(val createOrder: CreateOrder,
                    val getOrders: GetOrders,
                    val getOrderStatus: GetOrderStatus,
                    val deleteOrder: DeleteOrder,
                    val deliverOrder: DeliverOrder,
                    val payOrder: PayOrder) {

    @PostMapping(produces = arrayOf("application/hal+json"))
    @ResponseStatus(HttpStatus.CREATED)
    fun createOrder(@RequestBody createOrderRequest: CreateOrderRequest) : CreateOrderResponseBody {
        return createOrder.create(createOrderRequest) {
            it.toResponseBody()
        }
    }

    @GetMapping
    fun getOrders() : List<GetOrdersResponseBody> {
        return getOrders.getOrders() {
            it.map { it.toResponseBody() }
        }
    }

    @GetMapping("/{orderId}/status")
    fun getOrderStatus(@PathVariable orderId: String): String {
        return getOrderStatus.getStatus(GetOrderStatusRequest(orderId)) {
            it.status.name.toLowerCase()
        }
    }

    @PostMapping("/{orderId}/payment")
    fun payForOrder(@PathVariable orderId: String) {
        payOrder.pay(PayOrderRequest(orderId)) {}
    }


    @DeleteMapping("/{orderId}")
    fun deleteOrder(@PathVariable orderId: String) {
        deleteOrder.delete(DeleteOrderRequest(orderId))
    }

    @PostMapping("/{orderId}/delivery")
    fun deliverOrder(@PathVariable orderId: String) {
        deliverOrder.deliver(DeliverOrderRequest(orderId))
    }
}

data class HalLink(val href: String)

data class CreateOrderResponseBody(val id: String, val customer: String, val amount: BigDecimal, val _links: Map<String, HalLink>)
fun CreateOrderResponse.toResponseBody(): CreateOrderResponseBody {
    val links = mapOf(Pair("status", HalLink("/${id}/status")))
    return CreateOrderResponseBody(id, customer, amount, links)
}

data class GetOrdersResponseBody(val id: String, val customer: String, val status: String)
fun GetOrdersResponse.toResponseBody() : GetOrdersResponseBody {
    return GetOrdersResponseBody(id, customer, status.name.toLowerCase())
}
{% endhighlight %}

Again, you can see a couple of interesting pieces here. For example, the POST of `/order` actually directly uses the request object of the use case. If we wanted to be completely decouples, we should copy that data structure into a separate one, but in this case, it would be a one-to-one copy. Re-using the request object will save you some code and time here, but off course this comes at a cost. If you wanted to change the request of the use case, your REST endpoint will suddenly be impacted as well. It's a compromise, but as long as you're aware of the dangers, it's a valid compromise you can make in this case.

Here you also see the higher-order functions in action. They are able to map the response models from the use case into REST specific datastructures. For example, the response of the creation of an order also adds some HAL hypermedia links. 

If you want to run this, you'll have to provide some mock implementations of the use cases for now. For example, you can (for now) provide the following implementations in your application layer:

{% highlight kotlin %}
@UseCase
class MockCreateOrder : CreateOrder {
    override fun <T> create(request: CreateOrderRequest, presenter: (CreateOrderResponse) -> T): T {
        return presenter(CreateOrderResponse("test", request.customer, BigDecimal.TEN))
    }
}

@UseCase
class MockGetOrders : GetOrders {
    override fun <T> getOrders(presenter: (List<GetOrdersResponse>) -> T): T {
        return presenter(listOf(GetOrdersResponse("test", "John Doe", Status.OPEN, listOf())))
    }
}

@UseCase
class MockGetOrderStatus : GetOrderStatus {
    override fun <T> getStatus(request: GetOrderStatusRequest, presenter: (GetOrderStatusResponse) -> T): T {
        return presenter(GetOrderStatusResponse(Status.OPEN))
    }
}

@UseCase
class MockDeliverOrder : DeliverOrder {
    override fun deliver(request: DeliverOrderRequest) {
    }
}

@UseCase
class MockDeleteOrder : DeleteOrder {
    override fun delete(request: DeleteOrderRequest) {
    }
}

@UseCase
class MockPayOrder : PayOrder {
    override fun <T> pay(request: PayOrderRequest, presenter: (PayOrderResponse) -> T): T {
        return presenter(PayOrderResponse(Status.PAID))
    }
}
{% endhighlight %}

And add some component scanning magic in your main partition to make sure Spring can handle your custom annotions:

{% highlight kotlin %}
@Configuration
@ComponentScan(basePackages = ["be.sourcedbvba.restbucks.usecase"],
        includeFilters = [ComponentScan.Filter(type = FilterType.ANNOTATION,
        value = [UseCase::class])])
class UseCaseConfiguration
{% endhighlight %}

Running this application will yield you the REST API you can already consume. Next up: implementing the application layer and the domain.


