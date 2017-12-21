---
layout: post
category : article
title: "Java 9 modules and Spring Boot? I give up"
comments: true
tags : [technical]
---

Java 9 is here, whether we like it or not. And because of that reason, I decided to do a deep dive into Java 9 and more particularly the Java 9 module system known as JPMS. But after weeks of struggling, I'm throwing in the towel. In my opinion, it's just not worth it to subject yourself to the amount of pain the migration to the Java 9 module path entails. But here are some of the things I encountered.

## Gradle and JPMS? Eeeehhhh....

I build all my sources using Gradle. However, although building Java 9 modules is documented, it is by no means an easy path. You'll have to do quite a bit of buildscript tinkering (and in my case serious re-ordering) in order to get everything to work as you want it to. One major gripe: if you follow the documented path, you'll have to duplicate your module name in both the module descriptor (`module-info.java`) and your buildscript...
Eventually, that was the least of my concerns, fixing this was relatively easy and reasonably documented.

## The frickin' transaction API

Anyone that has built a Spring Boot application that writes to a database has probably used `@Transactional` before. It's part of the standard EE API's and therefor you'd think using it would be a breeze. Well... it isn't. See, in order to use that annotation, you need to use the `javax.transaction:javax.transaction-api:1.2` dependency. No biggie, I'll add this to the dependencies. Everything compiles, yay! Now you run your Spring Boot application, which requires you to add the `java.sql` module because for some reason it uses `java.sql.SQLException` and off course, the runtime breaks because of this. Why? Because the `java.sql` module exports the `javax.transaction.xa` package, as does the dependency you just added. Split package dependencies are prohibited using the module path, so you're out of luck. Oh well, no annotation-based transactional use cases running with the Java 9 module path, I guess? 
"But you're using Spring Boot? Why not just use the @Transactional from them?", you might say. Well, I'm also trying to adhere to Clean Architecture, which means no Spring in my application layer (where I would define transactional boundaries). Any technology that forces me to compromise on CA principles causes serious concern to me.

## Spring Boot 2 just isn't Java 9 ready (yet?)

Granted, Java 9 may work perfectly if you build basic, simple apps that adhere to the rules that are enforced by Java 9. But Spring, as it stands, it currently the most used enterprise framework out there. And the sad reality is that it's just unusable with the module path as it currently is (I tested with 5.0.2). You need to use a couple of `--add-opens` to your runtime JVM arguments and be prepared to add `opens ...` just about everywhere in your application that exposes something to the Spring context that isn't exported yet. It's a very painful experience to say the least and in hindsight, a futile effort and a waste of time.
But to be honest: Juergen Hoeller warned us about this. He explicitly said that Spring would compile on the module path but wouldn't make any guarantees that it would run on the module path. And he was right: it doesn't. 

{% highlight text %}
Caused by: org.springframework.aop.framework.AopConfigException: Unable to instantiate proxy using Objenesis, and regular proxy instantiation via default constructor fails as well; nested exception is java.lang.NoSuchMethodException: org.springframework.boot.autoconfigure.http.HttpMessageConverters$$EnhancerBySpringCGLIB$$1d90bff9.<init>()
        at spring.aop@5.0.2.RELEASE/org.springframework.aop.framework.ObjenesisCglibAopProxy.createProxyClassAndInstance(ObjenesisCglibAopProxy.java:82) ~[spring-aop-5.0.2.RELEASE.jar:na]
        at spring.aop@5.0.2.RELEASE/org.springframework.aop.framework.CglibAopProxy.getProxy(CglibAopProxy.java:205) ~[spring-aop-5.0.2.RELEASE.jar:na]
        ... 79 common frames omitted
Caused by: java.lang.NoSuchMethodException: org.springframework.boot.autoconfigure.http.HttpMessageConverters$$EnhancerBySpringCGLIB$$1d90bff9.<init>()
        at java.base/java.lang.Class.getConstructor0(Class.java:3322) ~[na:na]
        at java.base/java.lang.Class.getDeclaredConstructor(Class.java:2510) ~[na:na]
        at spring.aop@5.0.2.RELEASE/org.springframework.aop.framework.ObjenesisCglibAopProxy.createProxyClassAndInstance(ObjenesisCglibAopProxy.java:76) ~[spring-aop-5.0.2.RELEASE.jar:na]
        ... 80 common frames omitted
{% endhighlight %}

And yes, Spring Boot still is being worked on. But as these issues start from the base of the Spring framework, it's a fair bet to state that Spring Boot 2's final version will have the same issues that I encountered. 

## Even IntelliJ doesn't play nice

The java.se.ee module was the bane of my existence for a couple of days and to some extent, still is. This piece of infernal deviance, deprecated by the way, not only requires the `java.transaction` module (which exports the root `javax.transaction` package, see above for the issues that causes), but also the `java.xml.ws.annotation` module, which exposes `javax.annotation`, home of annotations like `@PostConstruct` and heavily used in Spring. IntelliJ however, for some reason, insists on using this module even if you have the `javax.annotation:javax.annotation-api:1.2` dependency. You can add the `requires javax.annotation.api` in your module descriptor and everything will compile (and run), although the IntelliJ editor will still show an error in the code. Even IntelliJ is making my life hell.

## Be prepared to exclude transitive dependencies

Remember the issues with split packages? Well, as it turns out, there are quite a few libraries out there that shade existing API's or repackage them. In pre-Java 9 this wouldn't be that much of an issue, the runtime would just pick one and move along. But now the compiler will nag about this. To be fair, it's not Java 9's fault that things like JBoss Undertow don't use the correct dependencies and that there are multiple ways to have @PostConstruct on your classpath. But it's a real drag that you have to compile, see an error on split packages, remove, compile again and repeat the cycle until compilation works. JBoss seems to be a champion in causing things like this. 

## My advise

If you're building a Spring Boot application, I advise you compile with the module path if possible (and relatively painless) and run with the standard classpath. Don't shoot yourself in the foot trying to get Spring Boot running on the Java 9 module path. Compiling with the module path already has some advantages and kind of forces you to keep a bit more track of your dependencies, so I think there is value in using (or trying to use) it. Also being forced to think about which packages you export to the outside world and which packages should remain hidden as internal implementation is a nice exercise and might benefit your codebase. 
That being said, even tools like IntelliJ seem to have some issues when it comes to the module path, so YMMV. 

All in all, Java 9 modules are probably something people will start using when Java 10 or 11 arrives. At the moment, it's painful, obtrusive and just plain annoying at times. Nothing should be this painful to adopt, especially a core feature of a new Java release. Even wildcards in Java 5 or the lambda's in Java 8 gave me less of a headache.
