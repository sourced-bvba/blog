---
layout: post
category : article
title: "Implementing microservices using KumuluzEE"
comments: true
tags : [java]
---

A lot of people in the Java ecosphere know about writing embedded microservices (without a separate container) using either Spring Boot or Dropwizard. Both have their merits but neither is completely Java EE oriented. Both reuse some Java EE component, for example Dropwizard uses JAX-RS for REST APIs and both support JPA for persistence. But if you want to use pure Java EE this becomes quite hard (although Spring Boot can be used completely in a Java EE-style).

Recently some new tools have popped up to support Java EE microservices with embedded containers, such as JBoss Swarm, Payara Micro and KumuluzEE. Today, I’ll show how to set up a simple KumuluzEE project. Bear in mind that KumuluzEE is not final and does not support all the Java EE specifications, but for most microservices it does the job.

We’re going to start off adding the necessary dependencies to our build file (Gradle).

``` groovy
compile 'com.kumuluz.ee:kumuluzee-core:1.0.0'      
compile 'com.kumuluz.ee:kumuluzee-servlet-jetty:1.0.0'      
compile 'com.kumuluz.ee:kumuluzee-cdi:1.0.0'      
compile 'com.kumuluz.ee:kumuluzee-jax-rs:1.0.0'      
compile 'com.kumuluz.ee:kumuluzee-bean-validation:1.0.0'
```

This adds the necessary dependencies to build a basic REST based microservice.

Next, we’re adding the basic Application class needed for JAX-RS deployments.

``` java
@ApplicationPath("/rest/")      
public class TestApplication extends Application {      
}
```

This will set the root path for all the REST endpoints you’ll add. Next, we’re going to add a basic REST endpoint.

``` java
@Path("/test")      
public class TestResource {                 
  @GET          
  public int getInt() {              
    return 42;          
  }      
}
```

To run the application, you can start it in your IDE using the EeApplication class from KumuluzEE. This class has a main method and will start up the embedded Jetty container and deploy all the EE classes. Nothing more is needed. Alternatively, you can create an uberjar or use Capsule to create a self-running JAR, using the EeApplication as the main class.

That’s all there is to it. KumuluzEE supports the CDI standard, so you can easily add any of the Deltaspike integrations. It also supports JPA out of the box. It does not support JSF or WebSockets yet, but this is to be included in the next version. However, nothing prohibit you to include for example Atmosphere to add support for WebSockets already.

While not as feature-complete as the other players out there (Dropwizard and Spring Boot), KumuluzEE is a welcome addition to the microservices platforms.
