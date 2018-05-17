---
layout: post
category : article
title: "Tackling Restbucks with Clean Architecture, episode 4"
comments: true
tags : [technical]
---

[Part 1 here]({% post_url 2018-02-25-tackling-restbucks-clean-architecture %})
[Part 2 here]({% post_url 2018-03-04-tackling-restbucks-clean-architecture.2 %})
[Part 3 here]({% post_url 2018-03-05-tackling-restbucks-clean-architecture.3 %})

It took a while, but here's the last installment of the series on how to implement a RESTful application, using Restbucks as the example.

In the previous examples, I've shown you how you can implement the entire service from the REST controller to the backing database, using a domain and use cases in betweem. But some of the limitations of Clean Architecture make certain aspects of an application not so straightforward at first glance. Let's take transactionality for example.

In standard Spring applications, you'd probably put a dependency on `spring-tx` somewhere in your code. The problem with our approach is that you probably want to have your application API as the transactional boundary of your application. As you may remember, the application layer or domain layer should not have any technical framework dependencies and Spring is no exception there. 

So how do we solve this conundrum?

Why, AOP off course! Aspects allow to write concerns in different modules, which is ideal for this use, and allow us to defer these concerns to the edge of our appliction, which is exactly where we want our technical framework dependencies to be. So we build a new infrastructure layer, let's call it `infra-transaction`. 

The new transaction layer will depend on 3 things: the application API layer (so that it can access the annotation), the Spring transaction API (`spring-tx`) and the necessary libraries for AOP (I'm using `spring-boot-starter-aop`). We start off by writing an aspect.

{% highlight kotlin %}
@Aspect
class TransactionalUseCaseAspect(private val transactionalUseCaseExecutor: TransactionalUseCaseExecutor) {

    @Pointcut("@within(useCase)")
    fun inUseCase(useCase: UseCase) {
    }

    @Around("inUseCase(useCase)")
    fun useCase(proceedingJoinPoint: ProceedingJoinPoint, useCase: UseCase): Any? {
        return transactionalUseCaseExecutor.executeInTransaction(Supplier { proceedingJoinPoint.proceed() })
    }
}
{% endhighlight %}

This will match all the public methods in the use cases of the application, which is what we want. This aspect uses a `TransactionalUseCaseExecutor` which looks like this.

{% highlight kotlin %}
open class TransactionalUseCaseExecutor {
    @Transactional
    open fun <T> executeInTransaction(execution: Supplier<T>): T {
        return execution.get()
    }
}
{% endhighlight %}

This class is annotated with Spring's `@Transactional`, so it will correctly start a transaction. You could choose to use a lower level approach and use `TransactionTemplate`, but in this case, this approach is sufficient.

To finish, I create a Spring configuration file to define the beans and necessary functionality in order to get the AOP working.

{% highlight kotlin %}
@Configuration
@EnableAspectJAutoProxy
@EnableTransactionManagement
class UseCaseTransactionConfiguration {
    @Bean
    fun useCaseTransactionAspect(transactionTemplate: TransactionalUseCaseExecutor) = TransactionalUseCaseAspect(transactionTemplate)

    @Bean
    fun transactionalUseCaseExecutor() = TransactionalUseCaseExecutor()
}
{% endhighlight %}

If we add this module to the main partition and start the application, the use cases with the `@UseCase` annotation will now be transactional, without even having touched the use-case classes! 

But we can do more with this, for example validation. What if we wanted to validate the argument of the use case (since we use a request/response approach, we assume here that a use case has either a single argument or none)? Then we can write an aspect like this:

{% highlight kotlin %}
@Aspect
internal class UseCaseValidatonAspect(val validator: Validator) {

    @Pointcut("@within(useCase)")
    fun inUseCase(useCase: UseCase) {
    }

    @Around("inUseCase(useCase)")
    fun useCase(proceedingJoinPoint: ProceedingJoinPoint, useCase: UseCase): Any? {
        if(proceedingJoinPoint.args.size > 1) {
            validateUseCaseArgument(proceedingJoinPoint.args[0]);
        }
        return proceedingJoinPoint.proceed()
    }

    private fun validateUseCaseArgument(arg: Any) {
        val validate = validator.validate(arg)
        if(validate.isNotEmpty()) {
            throw ConstraintViolationException(validate)
        }
    }
}
{% endhighlight %} 

However, this may require you to add the `annotation-api` dependency on your application layer. It's not really a technical dependency, as the library solely consists of annotations, but if you truly want to decouple your validation definition, you'd have to find a framework that allows you to do so independently from the class it's validating. I actually made [something like this](https://gitlab.com/lievendoclo/kval-dsl) because of this very reason. 

In short, aspects and AOP allow you to decouple functionality from the layers and defer technical decisions to somewhere outside of your application or domain. 

I hope this series has shown you that you can easily write a RESTful application that adheres to clean architectural principles and that you don't need to [couple your REST to your backend with a single model](https://github.com/olivergierke/spring-restbucks). 

