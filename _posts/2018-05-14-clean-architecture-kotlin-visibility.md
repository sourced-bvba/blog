---
layout: post
category : article
title: "On clean architecture, Kotlin and visibility"
comments: true
tags : [kotlin]
---

Let me start off by saying that I really like Kotlin. To me, it's the language that Java was meant to be, providing a plethora of features and all the power of the JVM. In addition, I like the concept of clean architecture, something I have talked about in lengths in previous articles. 

Recently I had another look at Simon Brown's 2016 Devoxx talk about building modular monoliths. If you're already accustomed to clean architecture, you're familiar with the notion of modules: you'll probably have separate modules for your application, domain and infrastructure layers. At the end of the presentation, Simon gave one specific piece of advice.

> Stop making everything public

This little piece of advice, and quite a few others from that talk, got me thinking about how I structure my applications. To this day, I still tend to layer my packages into their structural counterparts like `be.sourcedbvba.someapp.somedomain.usecase` and `be.sourcedbvba.someapp.somedomain.controller`. And I'm starting to realize this isn't really the right way forward. In addition, a lot of us tend to make a lot of things public. This is mainly due to what I just said, by creating structural layers in our packages, which makes package protected classes a lot harder to use. 

When you start to use Kotlin, you come across another hurdle. By default, Kotlin makes everything public. In addition, Kotlin does not have the concept of package protection, it only has 2 visibility modifiers: `private` and `internal`. But it made me think how much I could make `internal`, and as I'll show you, it so happens to be quite a lot, really.  In short, you can just about make every class besides your application API and domain `internal`. If you use Spring and classpath scanning, that is (otherwise each module will need to have at least one public class that exposes the components of those module to the outside world). 

For those not familiar with `internal`, it makes a class only visible for other classes which it's being compiled with. In essence: it's only visible at compile time within the same module. So if module X depends on module Y, and module X contains an `internal` class A, only classes in module X can use class A at compile time. 

If you look at the code in a typical clean architecture project, it's apparent that none of the classes inside a infrastructure layer should ever be accessible outside of their own modules. In addition, the `internal` modifier also disables the usage of the implementation of your application layer implementation outside of that module. In essence, with Kotlin, you may think you don't really need to physically separate the implementation from the API in the application layer, the `internal` modifier should take care of the rest. But don't do this. This would mean your application API would depend on your domain layer and therefor that dependency would trickle down transitively into your application layers, which is bad. Offcourse, you could set the dependency on `compileOnly` in Gradle or `provided` in Maven to mitigate this, but I still recommend physically separating the API and the implementation.

The concept of package by component, for example `be.sourcedbvba.someapp.somecomponent`, doesn't really apply to me when you're using Kotlin, because of the fact that it doesn't have a package visibility modifier, which makes the entire thing quite useless. But making everything public isn't great either and by using the `internal` modifier, you can limit the visibility of your classes in a modular application quite nicely. 

With Java 9 true modules, you can certainly achieve a higher level of granularity, but with `internal`, you can go a long way in order to ensure some modular encapsulation. If you haven't seen Simon's talk on modular monolith, be sure to have a look at it, you can find it on YouTube. 



