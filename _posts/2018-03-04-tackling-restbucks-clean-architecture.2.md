---
layout: post
category : article
title: "Tackling Restbucks with Clean Architecture, episode 2"
comments: true
tags : [technical]
---

[Part 1 here]({% post_url 2018-02-25-tackling-restbucks-clean-architecture %})

In the first part we tackled creating the necessary use cases and the REST interface for the Restbucks application. In this episode we're going to implement the application layer and the communication with the domain layer. The domain layer is where most of of the logic of your application will reside. It enforces the business rules of your domain and defines the data structures and behavior of that domain. The implementation role of the application layer is to retrieve domain entities and perform the necessary domain operations in order to fulfill the use case.

In this example, we'll use events to signal the wanted domain behavior, so we'll end up with an event-driven application. This will allow us to decouple the domain behavior from the implementation details later on. For example, instead of saving an entity through a repository, we'll send an `SomeEntitySaved` event, which can then be handled later on in the infrastructure layer handling the persistence. This is one way of doing it, the more direct approach is to have interfaces in the domain layer that are implemented later on in the persistence layer. How you choose to build your application is completely up to you.

The domain in this example is quite simple. You have an order with items and there are a couple of actions that can be performed on the domain object that can change the state of the domain object or invoke some sort of a behavior.

{% highlight kotlin %}
class Order(val id: String, val customer: String, status: Status, val items: List<OrderItem>) {
    lateinit var cost : BigDecimal
    var status : Status = status
        private set

    fun calculateCost() {
        cost = BigDecimal(Random().nextInt(20))
    }

    fun create() {
        calculateCost()
        return OrderCreated(this).sendEvent()
    }

    fun delete() {
        return OrderDeleted(id).sendEvent()
    }

    fun pay() {
        if(status == Status.OPEN) {
            status = Status.PAID
            return OrderPaid(id).sendEvent()
        } else {
            throw IllegalStateException("Order should be open in order to be paid")
        }
    }

    fun deliver() {
        if(status == Status.PAID) {
            return OrderDelivered(id).sendEvent()
        } else {
            throw IllegalStateException("Order has not been paid yet")
        }
    }
}

data class OrderItem(val product: String, val quantity: Int, val size: Size, val milk: Milk)
{% endhighlight %}

As you can see, this domain object's behavioral methods don't only change their state and check the logic, but also send events at appropriate. So from an event perspective, there are 4 events that can happens in the system.

{% highlight kotlin %}
data class OrderCreated(val order: Order) : DomainEvent 
data class OrderDeleted(val id: String) : DomainEvent
data class OrderDelivered(val id: String) : DomainEvent
data class OrderPaid(val id: String) : DomainEvent
{% endhighlight %} 

With regards to using some sort of event infrastructure framework, there are a couple of options. Axon for example is a quite good framework for building an event-driven application. I however use a very simple implementation that does most of the heavy lifting.

{% highlight kotlin %}
interface DomainEvent {
    fun sendEvent() {
        EventPublisher.Locator.eventPublisher.publishEvent(this)
    }
}

interface DomainEventConsumer<E : DomainEvent> {
    fun consume(event: E)
}

interface EventPublisher {
    fun publishEvent(event: DomainEvent)

    object Locator {
        lateinit var eventPublisher: EventPublisher
    }
}
{% endhighlight %}

This way my event publishing can also be nicely decoupled from any of the technical frameworks and once again deferred to an infrastructure layer. To give an example, if you wanted to use the Spring event mechanism, you can use the following classes to implement that support.

{% highlight kotlin %}
@Component
class SpringApplicationEventPublisher(private val applicationEventPublisher: ApplicationEventPublisher) : EventPublisher {
    override fun publishEvent(event: Any) {
        applicationEventPublisher.publishEvent(event);
    }
}

@Component
class EventPublisherHolderPopulator(private val eventPublisher: EventPublisher) : InitializingBean  {
    override fun afterPropertiesSet() {
        EventPublisher.Locator.eventPublisher = eventPublisher
    }
}

@Commponent
class SpringDomainEventConsumerRegistrar(val applicationEventMulticaster: ApplicationEventMulticaster,
                                         val eventConsumers: List<DomainEventConsumer<*>>) : InitializingBean {
    override fun afterPropertiesSet() {
        eventConsumers.forEach(this::registerEventConsumer)
    }

    private fun registerEventConsumer(it: DomainEventConsumer<*>) {
        val klass = determineEventClass(it)
        applicationEventMulticaster.addApplicationListener(EventConsumerListener(klass, it))
    }

    private fun determineEventClass(eventConsumer: DomainEventConsumer<*>): KClass<*> {
        return eventConsumer::class.members.first { it.name == "consume" }.parameters.get(1).type.classifier as KClass<*>
    }


    class EventConsumerListener<E : DomainEvent>(val klass: KClass<*>, val domainEventConsumer: DomainEventConsumer<E>) :
            ApplicationListener<PayloadApplicationEvent<E>> {
        override fun onApplicationEvent(event: PayloadApplicationEvent<E>?) {
            if(klass.isInstance(event!!.payload)) {
                domainEventConsumer.consume(event.payload)
            }
        }
    }
}
{% endhighlight %}

However, you can just create a very naive implementation to make sure everything still starts.

{% highlight kotlin %}
@Configuration
@ComponentScan
class DomainConfiguration {

    @PostConstruct
    fun initPublisher() {
        EventPublisher.Locator.eventPublisher = MockEventPublisher()
    }
}

class MockEventPublisher : EventPublisher {
    override fun publishEvent(event: DomainEvent) {
    }
}
{% endhighlight %}

The last part of the domain layer is a way to retrieve domain entities in our application layer, the gateway. In this example, the gateway doesn't need to retrieve much in order to achieve the use case goals (just enough will do).

{% highlight kotlin %}
interface OrderGateway {
    fun getOrders() : List<Order>
    fun getOrder(orderId: String) : Order
}
{% endhighlight %}

Now that we have our domain implemented, we can use this to implement the use cases we defined earlier. As most of the behavior is contained inside the domain layer, this implementation is very straightforward. The most elaborate ones are CreateOrder and GetOrders, as they require some mapping between the domain model and the response model.

{% highlight kotlin %}
@UseCase
class CreateOrderImpl : CreateOrder {
    override fun <T> create(request: CreateOrderRequest, presenter: (CreateOrderResponse) -> T): T {
        val order = request.toOrder()
        order.create()
        return presenter(order.toResponse())
    }

    fun CreateOrderRequest.toOrder() : Order {
        val id = UUID.randomUUID().toString()
        return Order(id, customer, Status.OPEN, items.map { it.toOrderItem() })
    }

    fun CreateOrderRequestItem.toOrderItem(): OrderItem {
        return OrderItem(product, quantity, size, milk)
    }

    fun Order.toResponse() : CreateOrderResponse {
        return CreateOrderResponse(id, customer, cost)
    }
}

@UseCase
class GetOrdersImpl(val orderGateway: OrderGateway) : GetOrders {
    override fun <T> getOrders(presenter: (List<GetOrdersResponse>) -> T): T {
        val orders = orderGateway.getOrders()
        return presenter(orders.map { it.toResponse() })
    }

    fun Order.toResponse() : GetOrdersResponse {
        return GetOrdersResponse(id, customer, status, items.map { it.toResponse() })
    }

    fun OrderItem.toResponse() : GetOrdersResponseItem {
        return GetOrdersResponseItem(product, quantity, size, milk)
    }
}
{% endhighlight %}

As you can see, I'm also using Kotlin's extension method mechanism here. It makes the code a bit nicer to read, in my opinion, but feel free to choose you own style that suits you best. The other use cases merely get the required domain instance and call a method on it, for example.

{% highlight kotlin %}
@UseCase
class DeliverOrderImpl(val orderGateway: OrderGateway) : DeliverOrder {
    override fun deliver(request: DeliverOrderRequest) {
        val order = orderGateway.getOrder(request.orderId)
        order.deliver()
    }
}

@UseCase
class PayOrderImpl(val orderGateway: OrderGateway) : PayOrder {
    override fun pay(request: PayOrderRequest) {
        val order = orderGateway.getOrder(request.orderId)
        order.pay()
    }
}
{% endhighlight %}

The only thing that's missing to get the application to run once again is a mock implementation of the `OrderGateway`. 

{% highlight kotlin %}
@Component
class MockOrderGateway : OrderGateway {
    override fun getOrder(orderId: String): Order {
        return Order("one", "John Doe", Status.OPEN, listOf())
    }

    override fun getOrders(): List<Order> {
        return listOf(
                Order("one", "John Doe", Status.OPEN, listOf()),
                Order("two", "Jane Doe", Status.OPEN, listOf())
        )
    }
}
{% endhighlight %}

So now we have implemented the web layer, the entire application layer and the domain layer. All that's left is implementing the persistence layer, which I'll cover in the next episode.

[Part 3 here]({% post_url 2018-03-05-tackling-restbucks-clean-architecture.3 %})
[Part 4 here]({% post_url 2018-05-17-tackling-restbucks-clean-architecture.4 %})