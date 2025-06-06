---
layout: post
category : article
title: "Using JAX-RS with Spring Boot instead of MVC"
comments: true
tags : [java]
---

Spring users that frequently write REST APIs are very familiar with the Spring MVC REST support to write their endpoints. Java EE users however are more accustomed to the JAX-RS specification. It's however really easy to integrate JAX-RS into Spring applications, especially Spring Boot applications. <!--more-->

## Why

But why would you do this? Spring MVC should be enough, right? Well, there's something to be said for the cleanliness that is inherent to the JAX-RS spec. Whereas the Spring MVC approach is sufficient, JAX-RS is targeted solely to implementing REST APIs. Spring MVC isn't built for that purpose alone and personally, I think it shows. Sometimes it can be a bit verbose or confusing.

For example, if you omit a method on a Spring MVC REST controller, Spring MVC happily assumes you want to have this endpoint available for all HTTP methods. For MVC purposes, this might be okay, for REST, this is not. And if you add the method to the `@RequestMapping`, you suddenly can't use the terse notation and need to explicitly declare the `value` parameter on the notation. Same thing with setting explicit MIME types for an andpoint. In that case, I really like the `@GET`, `@POST`, `@Produces` and `@Consumes` parameters the JAX-RS API has. You may end up with more annotations, but at least everything is clear and defined explicitly.

At the moment, I haven't found anything that Spring MVC can do for REST which JAX-RS couldn't.

## Enabling Jersey in Spring Boot

First you need to add a dependency to your application.

``` groovy
compile "org.springframework.boot:spring-boot-starter-jersey"
```

The only thing you need to do then is to add a Jersey `ResourceConfig` class to your Spring context.

``` java
@Component
public class JerseyConfig extends ResourceConfig {
    public JerseyConfig() {
        registerEndpoints();
    }

    private void registerEndpoints() {
        // register(...);
    }
}
```

Now you can start writing JAX-RS endpoint classes. Each endpoint class must be a Spring bean in order to be able to use Spring DI inside the JAX-RS endpoints. For example, this is a very simple endpoint:

``` java
@Component
@Path("/hello")
public class HelloWorldEndpoint {
    @GET
    @Path("/world")
    public String test() {
        return "Hello world!";
    }
}
```

When you register this endpoint in your `ResourceConfig` class (`register(HelloWorldEndpoint.class);`) the endpoint will be available, in this case on `/hello/world`.

## Additional things

The Jersey integration in Spring Boot enables you to use every feature that Jersey uses, such as `@Provider` annotated classes. Just remember to register them in the `ResourceConfig` class, Spring won't pick these up automatically like standard Java EE does. You can use some classpath scanning trickery, auto-registering beans with certain JAX-RS annotations, or you can choose to register entire packages (using `packages(...);` in your `ResourceConfig`).

With Jersey, you also get some very nice features for free, such as built-in WADL support. You just need to register the `WadlResource` class and you're good to go. Another feature I love in JAX-RS is the concept of `@BeanParam`. You can declare entire beans for your endpoint parameters and annotate the fields with `@HeaderParam`, `@QueryParam`, ... which in some cases can make life a lot easier and your endpoints a lot more readable and terse. Have a look at the Jersey documentation if you want to see what else you're able to use.

## Bonus: adding Swagger support

Swagger is quickly turning into the documentation framework of choice for REST APIs. Spring users that want to use Swagger can use SpringFox to do the integration, but Swagger has built-in support for JAX-RS. There are a couple of things you need to do.

First you need to add the dependency.

``` groovy
compile 'io.swagger:swagger-jersey2-jaxrs:1.5.3'
```

Second, you need to configure the `BeanConfig` for Swagger. You can do this in your `ResourceConfig` class.

``` java
BeanConfig beanConfig = new BeanConfig();
beanConfig.setVersion("1.0.2");
beanConfig.setSchemes(new String[]{"http"});
beanConfig.setHost("localhost:8080");
beanConfig.setBasePath("/");
beanConfig.setResourcePackage("your.resource.package.here,your.other.package.here");
beanConfig.setPrettyPrint(true);
beanConfig.setScan(true);
```

This `BeanConfig` can be configured with the needed information, most of which you can get out of Spring Boot's configuration, such as supported schemes, the port, the host, ...
Third, you need to register the Swagger resource.

``` java
register(ApiListingResource.class);
```

Now don't forget to annotate your resources with `@Api` so that Swagger will pick them up.

A complete `ResourceConfig` with WADL and Swagger support would look something like this:

``` java
public class JerseyConfig extends ResourceConfig {

    public JerseyConfig() {
        registerEndpoints();
        configureSwagger();
    }

    private void configureSwagger() {
        register(ApiListingResource.class);
        BeanConfig beanConfig = new BeanConfig();
        beanConfig.setVersion("1.0.2");
        beanConfig.setSchemes(new String[]{"http"});
        beanConfig.setHost("localhost:8080");
        beanConfig.setBasePath("/");
        beanConfig.setResourcePackage("your.resource.package.here,your.other.package.here");
        beanConfig.setPrettyPrint(true);
        beanConfig.setScan(true);
    }

    private void registerEndpoints() {
        register(WadlResource.class);
        register(HelloWorldEndpoint.class);
    }
}
```

## Conclusion

JAX-RS and Spring Boot are very easy to integrate. The choice for JAX-RS or Spring MVC REST endpoints is a personal one, both have the same features, some easier to implement in one or the other. Personally, for REST services, I like JAX-RS more. It's built for that purpose and the API reflects that choice in terms of ease of use.
