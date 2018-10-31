---
layout: post
category : article
title: "Another Spring Boot integration: Axon"
comments: true
tags : [java]
---

Creating Spring Boot integrations for frameworks is just so easy that I couldn't resist making another one. One of my recent posts covered CQRS principles and Axon is a framework that currently is the reference when it comes to implementing CQRS and event-sourcing in Java. As it happens it also has great integration with Spring.<!--more-->

Autoconfigurations with Spring Boot are actually just configurations with conditionals to get out of the way if you want to override the auto configuration. In the case of Axon, the autoconfiguration is very easy, defining an eventbus, eventstore, commandbus and gateway, in addition to the postprocessors than link annotated commandhandlers and eventhandlers to their buses.

So, without further ado, the configuration for adding Axon features to Spring Boot:

``` groovy
@Configuration
@ConditionalOnClass(CommandBus)
class AxonConfiguration {
    @Bean
    @ConditionalOnMissingBean
    CommandBus commandBus() {
         def bus = new SimpleCommandBus();
        return bus
    }

    @Bean
    @ConditionalOnMissingBean
    CommandGateway commandGateway() {
        def gateway = new DefaultCommandGateway(commandBus());
        return gateway
    }

    @Bean
    @ConditionalOnMissingBean
    EventBus eventBus() {
        def bus =  new SimpleEventBus()
        return bus
    }

    @Bean
    @ConditionalOnMissingBean
    EventStore eventStore() {
        def tempDir = File.createTempDir("axon", "_events")
        def resolver = new SimpleEventFileResolver(tempDir);
        def store =  new FileSystemEventStore(resolver)
        return store
    }

    @Bean
    AnnotationCommandHandlerBeanPostProcessor annotationCommandHandlerBeanPostProcessor() {
        def p =  new AnnotationCommandHandlerBeanPostProcessor()
        p.commandBus = commandBus()
        return p
    }

    @Bean
    AnnotationEventListenerBeanPostProcessor annotationEventListenerBeanPostProcessor() {
        def p = new AnnotationEventListenerBeanPostProcessor()
        p.eventBus = eventBus()
        return p
    }
}
```

If you now create a method annotated with `@CommandHandler` and send a command through the gateway, this configuration will make sure the method gets called.

You can always declare your own `EventBus`, `CommandBus`, `EventStore` or `CommandGateway` in your configuration and the autoconfigured beans will be ignored. So if you want the EventStore persisting events to a MongoDB or through JPA, you just need to create the beans in your config.

Currently, Spring Boot has a lot of integrations in its autoconfigure library but more can always be added. Axon seemed like a valid addition to that library (as is Hystrix covered in an earlier article). For example, Vaadin has also provided a library that provides Spring Boot integration. Another prime candidate would be Activiti. Spring Boot takes a lot of the guesswork out of integrating frameworks. If a framework has integration with Spring, you can make an autoconfiguration for Spring Boot. 
