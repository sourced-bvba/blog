---
layout: post
category : article
title: "Hystrix and Spring Boot"
comments: true
tags : [java]
---

Making your application resilient to failure can seem like a daunting task. Those who read "Release It!" know how many aspects there can be to making your application ready for the apocalypse. Luckily we live in a world where a lot of software needs such resilience and where there are companies who are willing to share their solutions.

Enter what Netflix has created: Hystrix.<!--more--> Hystrix is a Java library aimed towards making integration points less susceptible to failures and mitigating the impact a failure might have on your application. It provides the means to incorporate bulkheads, circuit breakers and metrics into your framework. Those not familiar with these concepts should read the book I mentioned earlier. For example, a circuit breaker makes sure that if a certain integration point is having trouble, your application will not be affected. If for example a integration point takes 20 seconds to reply instead of the normal 50ms, you can configure a circuit breaker that trips if 10 calls within 10 seconds take longer than 5 seconds. When tripped, you can configure a quick fallback or fail fast.

Hystrix has an elegant solution for this. Every command to an external integration point should get wrapped in a `HystrixCommand`. `HystrixCommand` provide support for circuit breakers, timeouts, fallbacks and other disaster recovery methods. So instead of directly calling the integration point, you'll call a command that in turn calls the integration point. Hystrix also allows you to choose whether you want to do this synchronously or asynchronously (returning a Future). 

One of the really nice things about Hystrix is that it also has support for metrics and even has a nice dashboard to show those metrics. I can almost imagine that every development team has this on the dashboard next to the Hudson/Jenkins monitor in the near future, just because it's so trivial to incorporate. 

Now, creating a new subclass for each and every distinct call to an integration endpoint may seems like a lot of work. It is, but the reasoning behind this is that incorporating Hystrix in your application should be explicit. However, if you really don't like this, Hystrix also supports Spring AOP and has a aspect that does most of the work for you, using a contributed module (javanica). The only thing you need to do is annotate the methods you want covered by Hystrix.

Whenever I see decent Spring integration, I now immediately look at Spring Boot support. Hystrix doesn't have autoconfiguration for Spring Boot yet, but it's really easy to implement. I used the annotation/aspect approach because I'm lazy and I like the transparency of going down this path.

First you need to add a couple of dependencies. Here's what you need in Gradle:

{% highlight groovy %}
compile("com.netflix.hystrix:hystrix-javanica:1.3.16")
compile("com.netflix.hystrix:hystrix-metrics-event-stream:1.3.16")
{% endhighlight %}

Then you need to create a configuration for Hystrix. I opted to create the configuration just like any other autoconfiguration module in Spring Boot (an `@Configuration` annotated class and a class describing the configuration properties). I also used conditional beans so that the .

{% highlight groovy %}
/**
 * {@link EnableAutoConfiguration Auto-configuration} for Hystrix.
 *
 * @author Lieven Doclo
 */
@Configuration
@EnableConfigurationProperties(HystrixProperties)
@ConditionalOnExpression("\${hystrix.enabled:true}")
class HystrixConfiguration {
    @Autowired
    HystrixProperties hystrixProperties;

    @Bean
    @ConditionalOnClass(HystrixCommandAspect)
    HystrixCommandAspect hystrixCommandAspect() {
        new HystrixCommandAspect();
    }

    @Bean
    @ConditionalOnClass(HystrixMetricsStreamServlet)
    @ConditionalOnExpression("\${hystrix.streamEnabled:false}")
    public ServletRegistrationBean hystrixStreamServlet(){
        new ServletRegistrationBean(new HystrixMetricsStreamServlet(), hystrixProperties.streamUrl);
    }
}

/**
 * Configuration properties for Hystrix.
 *
 * @author Lieven Doclo
 */
@ConfigurationProperties(prefix = "hystrix", ignoreUnknownFields = true)
class HystrixProperties {
    boolean enabled = true
    boolean streamEnabled = false
    String streamUrl = "/hystrix.stream"
}
{% endhighlight %}

In short, if you add this to your Spring Boot application, Hystrix will be automatically integrated in your application. As you might have seen, I've also added some configuration properties. I added support for the event stream that powers the dashboard and which is only activated if you add `hystrix.streamEnabled = true` to your `application.properties`. The URL through which the stream is served is also configurable (but has a sensible default). If you want, you can disable Hystrix as a whole by adding `hystrix.enabled = false` to your `application.properties`. This code is actually ready to be put into Spring Boot's autoconfigure module :).

Two simple classes and two simple dependencies and your code is ready for the apocalypse. Doesn't seem like a bad deal to me. Hystrix has a lot more to offer than I touched in this article (command aggregation, reactive calls through events, ...). If your application has a lot of integration points, certainly have a look at this library. Your application may be stable, but that doesn't mean that all the REST services you're calling are.
