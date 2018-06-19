---
layout: post
category : article
title: "What is a presenter?"
comments: true
tags : [technical]
---

Sometimes, there are days where you have serious technical discussions about the semantic meaning of a certain term. This week, I had one of those days.

I'm a big fan of Clean Architecture principles and if you look at the components that make up a clean architecture, there's something called a presenter, as seen in the image below.

![Clean Architecture Design](/img/CleanArchitectureDesign.png)

For most concepts, we have a clear understanding by now. Granted, the terms interactor and boundary may have been poorly named, which is why we use use cases and use case interfaces instead we discuss clean architecture between colleagues where I work. However, the presenter pattern proved to be a tough nut to crack. To some people, a presenter was used to present data to the outside world, being tightly coupled with the view technology. To other, presenters were the translation layer between the application layer (use cases) and the view, creating (or better said, mapping) a view model. This was the start of an interesting discussion, as to some mapping is not presenting and a presenter does way more than mapping.

When a presenter is interpreted as a mapper, the presenter has a return value and the use case mimics that return value. In Java, you would do this like this:

{% highlight kotlin %}
interface UseCase {
    <T> fun doSomething(presenter: Mapper<T>) : T
}

interface Mapper<T> {
    fun present(response: UseCaseResponse) : T
}

class UseCaseImpl {
    override <T> fun doSomething(presenter: Presenter<T>) : T {
        // do some stuff and get a UseCaseResponse
        return presenter.present(response)
    }
}
{% endhighlight %}

In other words, the use case would be called and it would map the response through the presenter, returning the presented value. 

{% highlight kotlin %}
class Controller(useCase: UseCase) {
    fun doSomething() : String {
        return useCase.doSomething({ it.value }) 
    }
}
{% endhighlight %}

In a lot of cases, this would probably work fine. However, the problem comes when you need to handle error situations. When using the approach above, you have a single return value and only one way to return it. What happens if there is a business exception? A presenter only handles the happy scenario...

Well, with Spring MVC you tend to write something like an exception handler, mapping the exception to something sensible. But here's the kicker: what if the exception is thrown by the domain and therefor isn't accessible from the web infrastructure layer? You suddenly can't write the exception handler anymore, because the exception is not on the classpath. Doh.

Uncle Bob wrote an interesting example called Hunt the Wumpus. This contained something called a receiver, something that could be called by the use case in order to notify the consumers of the result from different outcomes. Martin Fowler also described this pattern [here](https://martinfowler.com/articles/replaceThrowWithNotification.html). 

So what would our example look like?

{% highlight kotlin %}
interface UseCase {
    fun doSomething(presenter: Receiver)
}

interface Receiver {
    fun success(response: UseCaseResponse)
    fun failure(reason: String)
}

class UseCaseImpl {
    override fun doSomething(presenter: Receiver) {
        try {
            // do some stuff and get a UseCaseResponse
            presenter.success(response)
        } catch (ex: BusinessException) {
            presenter.failure(ex.reason)
        }
    }
}
{% endhighlight %}

The use case is no longer relying on exception in order to handle business error flows. The presenter contract clearly says which issues can happen and should be handled gracefully. Off course you can still have runtime exceptions, but those bubble up and get handled generically. You may also have noticed that there are no generics, nor are there return values in the interface or receive

On the controller side, you unfortunately now lose the ability to use lambdas. 

{% highlight kotlin %}
class JsonReceiver : Receiver {
    var result : ResponseEntity<*>
        private set

    override fun success(response: UseCaseResponse) {
        result = ResponseEntity.ok(JsonResult(response.value))
    }  

    override fun failure(reason: String) {
        result = ResponseEntity.status(HttpStatus.BAD_REQUEST).body(reason)
    }
}

class Controller(useCase: UseCase) {
    fun doSomething() : ResponseEntity {
        val receiver = JsonReceiver()
        useCase.doSomething(receiver) 
        return presenter.result
    }
}
{% endhighlight %}

There is a way to use lambdas if you pass through functions as parameters to the receiver and call those functions in the implementation, but I wouldn't recommend it as that approach has some serious drawbacks. I was going to add the implementation here as well, but in the end decided it really wasn't worth showing a bad idea.

So the question is, what is the correct usage of a presenter? Is it a mapper, or is it a receiver? Well, in my opinion, both. As Uncle Bob said in his tweet, the devil is in the details.

{% twitter https://twitter.com/unclebobmartin/status/1008746090584256512 %}

Sometimes it is a mapper, but sometimes it is a receiver. However, there is a slight difference in usage: mappers allow you to dictate a return value on your use case, receivers do not as you cannot know the return value from the different methods of the receiver.

So if both the mapper and the receiver can be aggregated under the concept of 'presenter', this makes the drawing regarding clean architecture a bit more complicated. Receivers have the added value of being able to see what could possibly go wrong (it's kind of a better checked exception, but without the dependency), but you sacrifice conveniency and brevity in your controller. Both approaches have their merits and to me, it depends on the situation to be able to decide which is the right one.

So in the end, is there a difference between a presenter and a mapper? Well, a mapper _is_ a presenter. However not all presenters _are_ mappers. They might do more. If they do, maybe we should give them a name, like receivers. This might be hard to put into a single drawing, as you will need to choose which pattern you will use in different circumstances. Presenter is not the superclass of Receiver or Mapper, it's more like a stereotype. Sometimes you'll use one, sometimes the other. But we're smart enough to decide which one to use. I hope. The devil is indeed in the details. But they do make for some really interesting discussions between developers.