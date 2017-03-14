---
layout: post
category : article
title: "Spring Boot and hypermedia, part 1: HAL"
comments: true
tags : [kotlin]
---

So must of us have already encountered hypermedia when dealing with REST API. Hypermedia is actually a bunch of metadata that allows you to create navigable or auto-discoverable REST services. In other words, by adding some data, you can navigate the resources within a system without actually having to know the exact URLS.

There are many implementations out there and the goals of this series is to show a couple of them, some of them more widely used than others. And while the goal of all the hypermedia implementations are the same, there is a big difference in their capabilities as you'll notice. In my examples, I'll be using Spring Boot and the code will be written in Kotlin for brevity.  

So first off is HAL, which is actually the hypermedia implementation supported out of the box by Spring in the Spring HATEOAS library. 

HAL (or Hypertext Application Language) is one of the more widely use hypermedia formats. It's simplistic in it's nature, only supporting links and embedded resources. 

An example HAL document would look like this:

{% highlight json %}
{
    "_links": {
        "self": { "href": "/orders" },
        "next": { "href": "/orders?page=2" },
        "ea:find": {
            "href": "/orders{?id}",
            "templated": true
        }
    },
    "currentlyProcessing": 14,
    "shippedToday": 20,
    "_embedded": {
        "ea:order": [{
            "_links": {
                "self": { "href": "/orders/123" },
                "ea:basket": { "href": "/baskets/98712" },
                "ea:customer": { "href": "/customers/7809" }
            },
            "total": 30.00,
            "currency": "USD",
            "status": "shipped"
        }, {
            "_links": {
                "self": { "href": "/orders/124" },
                "ea:basket": { "href": "/baskets/97213" },
                "ea:customer": { "href": "/customers/12369" }
            },
            "total": 20.00,
            "currency": "USD",
            "status": "processing"
        }]
    }
}
{% endhighlight %}

JSON documents that have been enriched by the HAL format have 3 items:
- The standard JSON state of your resource
- Links that point to other resources
- Embedded resources that may contain state and links of that embedded resource

We'll start off by creating some code that creates a basic REST endpoint that we can work on and that basically is able to return the raw JSON state without any hypermedia.

{% highlight kotlin %}
@@SpringBootApplication
@RestController
class HalApplication {
    @RequestMapping("/orders")
    fun getOrders() : Orders {
        return Orders(14, 20,
                listOf(Order(30.00, "USD", "shipped"), Order(20.00, "USD", "processing")))
    }
}

open class Orders(val currentlyProcessing: Int, val shippedToday: Int, val orders: List<Order>) 

open class Order(val total: Double, val currency: String, val status: String)

fun main(args: Array<String>) {
    SpringApplication.run(HalApplication::class.java, *args)
}
{% endhighlight %}

This will return a standard JSON document as we all know and love.

{% highlight json %}
{
   "shippedToday" : 20,
   "currentlyProcessing" : 14,
   "orders" : [
      {
         "total" : 30,
         "status" : "shipped",
         "currency" : "USD"
      },
      {
         "currency" : "USD",
         "status" : "processing",
         "total" : 20
      }
   ]
}
{% endhighlight %}

To add HAL support, both data classes need to extends `ResourceSupport` which enables them to add links. Now that we've done this we can add the links needed for the example. 

{% highlight kotlin %}
fun getOrders() : Orders {
	val firstOrder = Order(30.00, "USD", "shipped")
	firstOrder.add(Link("/orders/123", "self"))
	firstOrder.add(Link("/baskets/98712", "ea:basket"))
	firstOrder.add(Link("/customers/7809", "ea:customer"))
	val secondOrder = Order(20.00, "USD", "processing")
	secondOrder.add(Link("/orders/124", "self"))
	secondOrder.add(Link("/baskets/97213", "ea:basket"))
	secondOrder.add(Link("/customers/12369", "ea:customer"))
	val orders = Orders(14, 20,
			listOf(firstOrder, secondOrder))
	orders.add(Link("/orders", "self"))
	orders.add(Link("/orders?page=2", "next"))
	orders.add(Link(UriTemplate("/orders{?id}"), "ea:find"))
	return orders
}
{% endhighlight %}

I've hardcoded the links here, but using the `ControllerLinkBuilder` you can reference methods on a controller to get the link based on the `RequestMapping` value. But this article hardcoding them will suffice.

If we now get the JSON from the `/orders` endpoint, this is what we get.

{% highlight json %}
{
   "orders" : [
      {
         "currency" : "USD",
         "total" : 30,
         "status" : "shipped",
         "_links" : {
            "self" : {
               "href" : "/orders/123"
            },
            "ea:basket" : {
               "href" : "/baskets/98712"
            },
            "ea:customer" : {
               "href" : "/customers/7809"
            }
         }
      },
      {
         "currency" : "USD",
         "_links" : {
            "ea:basket" : {
               "href" : "/baskets/97213"
            },
            "ea:customer" : {
               "href" : "/customers/12369"
            },
            "self" : {
               "href" : "/orders/124"
            }
         },
         "total" : 20,
         "status" : "processing"
      }
   ],
   "currentlyProcessing" : 14,
   "_links" : {
      "ea:find" : {
         "href" : "/orders{?id}",
         "templated" : true
      },
      "next" : {
         "href" : "/orders?page=2"
      },
      "self" : {
         "href" : "/orders"
      }
   },
   "shippedToday" : 20
}
{% endhighlight %}

Almost there. The main difference now is the orders, which in the HAL format is an embedded resource. And here be dragons. The support for embeddeds in Spring HATEOAS isn't that great and despite sifting through the code and possible examples, I could get any of them to work. What I ended up doing is handling the embedded as a basic map. Perhaps it's due to the fact that I'm using Kotlin or I'm just missing something, frankly I don't know.

So now the `Orders` looks like this:

{% highlight kotlin %}
open class Orders(val currentlyProcessing: Int, val shippedToday: Int, val _embedded: Map<String, List<Any>>) : ResourceSupport()
{% endhighlight %}

and the creation of the `Orders` object looks like this:

{% highlight kotlin %}
val orders = Orders(14, 20, mapOf(Pair("ea:order", listOf(firstOrder, secondOrder))))
{% endhighlight %}

After this, you'll end up with the JSON example we started the article with. 

So what do I think of HAL and the Spring HATEOAS support? 

Personally, as far as linking is concerned it's very nice. With the `ControllerLinkBuilder` you can easily build up the link URLs which will stay in sync with the annotations. The embedded concept is something I'd steer clear of, because of the lack of documentation and clear rules of usage. I don't see the point in embedding a resources through a convoluted mechanism like that when you can either use the link system for it or just plain using a list or any other collection.

HAL also only supports GET operations, as there is no way to indicate whether a link is a GET, a POST or a PUT, so it assumes GET. It's principle usage is for discovering data within a REST based API, but to interact with the API regarding data manipulation you'll still need some other kind of documentation.