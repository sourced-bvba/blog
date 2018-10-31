---
layout: post
category : article
title: "Hystrix and Spring Boot's health endpoint"
comments: true
tags : [java]
---

In an [earlier post](2014-08-19-hystrix-spring-boot.md) I showed how easy it is to integrate Hystrix into a Spring Boot application. Now I'm going to show you a neat trick which combines the health indicator endpoint in Spring Boot and the metrics provided by Hystrix.<!--more-->

Hystrix has a built-in system to query the metrics that drive the framework. For example you can query the metrics of each command such as the mean execution time or whether the circuit breaker for that command has tripped. And it's that last one that is very interesting to the health indicator of your application.

Most production environments have a dashboard that show the health of an application's instances. If the circuitbreaker has tripped, your application is essentially in an unhealthy state. The circuit breaker mechanism will ensure that failures won't cascade, but in a clustered environment you'd want that server removed from the pool or at least have a general indication that something is wrong. 

Spring Boot's health endpoints works by querying various indicators. Like most things in Spring Boot, indicators are only active if there are components that can be checked. For example, if you have a datasource, an indicator will become active checking the state of that datasource. The same thing happens with NoSQL or AMQP connections. 

A simple implementation with Hystrix, which I'll show in a minute, could be that when there is a tripped circuitbreaker in the system, the health of the application might be 'out of service'. This is actually very easy to do. You just need to add a bean in your configurations returning an implementation of `AbstractHealthIndicator`:

{% highlight groovy %}
class HystrixMetricsHealthIndicator extends AbstractHealthIndicator {
    @Override
    protected void doHealthCheck(Health.Builder builder) throws Exception {
        def breakers = []
        HystrixCommandMetrics.instances.each {
            def breaker = HystrixCircuitBreaker.Factory.getInstance(it.commandKey)
            def breakerOpen = breaker.open?:false
            if(breakerOpen) {
                breakers << it.commandGroup.name() + "::" + it.commandKey.name()
            }
        }
        breakers ? builder.outOfService().withDetail("openCircuitBreakers", breakers) : builder.up()
    }
}
{% endhighlight %}

Whenever a circuitbreaker gets tripped, the health endpoint will return the state of the application as `OUT_OF_SERVICE` and will also return the name of the open circuit breakers (the command key and the group it's in).

Now, this implementation can go a whole lot further. For example, you can add a new state to the health indication, for example `UNSTABLE`. This will however require you to change to order of the health aggregator, as Spring Boot will aggregate all the indicators and show a single application state. The new state needs to be fit in the existing order of states (`DOWN` > `OUT_OF_SERVICE` > `UP` > `UNKNOWN`). In the case of `UNSTABLE`, it would probably be between `OUT_OF_SERVICE` and `UP`.

I can also think of a use-case in which the tripping of certain circuit breakers may be more critical than others, in which case the state of the application might really become `OUT_OF_SERVICE`. In that case you might decide to remove the instance from the pool of available instances (in a clustered environment) or restart the server. Or you can automate the process :).

The last use case I'll discuss is when your application is slow or is getting hammered by requests, which can be detected by Hystrix as well. In this case, you can introduce yet another state `STRUGGLING`, which would logically be between `UNSTABLE` and `UP`. In this case you can automate a process that starts up another instance and add it automatically to the pool. You can also see this the other way around, adding a state `UNUSED` which is on the same level as `UP`. This might indicate you have too many instances running and can possible shutdown that node (if it's not the only one), or that you need to take a look at the load-balancing.

As you can see, with such mechanisms it becomes possible to create a self-regulating instance pool, creating and removing instances as it goes. The health indicators in Spring Boot are an invaluable tool for DevOps teams and show how versatile Spring Boot actually is.

*UPDATE:*

Normally, if you want to alter the order in which statuses are aggregated, you can use a property in your `application.properties` like `health.status.order = DOWN,OUT_OF_SERVICE,UNSTABLE,STRUGGLING,UP,UNKNOWN` as documented. However, if you're using the YAML-style properties, you're out of luck, as there's [an annoying bug](https://jira.spring.io/browse/SPR-11759) that's restricting you from using this feature. So if you're using YAML properties, you'll have to configure the `HealthAggregator` yourself. Luckily, this isn't that hard, just add this bean to your application context:

{% highlight groovy %}
@Bean
HealthAggregator healthAggregator() {
    def healthAggregator = new OrderedHealthAggregator();
    healthAggregator.setStatusOrder(["DOWN", "OUT_OF_SERVICE", "UNSTABLE", "UP", "UNKNOWN"]);
    return healthAggregator;
}
{% endhighlight %}

Why they didn't use the `@EnableConfigurationProperties` in the `HealthIndicatorAutoConfiguration` is a mystery to me, as this would have solved the issue. Perhaps I'll do it myself and make a pull request.
