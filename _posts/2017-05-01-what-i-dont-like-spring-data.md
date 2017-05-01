---
layout: post
category : article
title: "Spring Data and Clean Architecture"
comments: true
tags : [opinion]
---

So recently I've ventured off my familiar back-end track and starting doing some basic front-end stuff again. Not the fancy SPA with Angular or React, but plain and simple MVC with Thymeleaf as my templating language. And I must say, since I last explored it, it hasn't changed that much but it's still very powerful and in my humble opinion more than enough for a lot of use cases. 

Thymeleaf also has some nice extensions. If your application is secured through Spring Security, enabling or disabling components based on authorities is quite simple. After 2 hours or so, using some Bootstrap magic, I was able to whip up a half-decent UI. So then I looked at what else I could find and so I found a small extension that used Spring Data's `Pageable` interface to add sorting and paging to a basic datatable. The demo looked nice and I started looking at the code.

Then reality hit me. This extension uses Spring Data's libraries at its core and therefor you need to depend on Spring Data on your front-end. Urgh #1. You see, Spring Data is inherently an infrastructure library, but normally not used on the front-end facing side of your hexagonal architecture. It also brings in a some bagage, like the core parts of Spring. In my front-end that's not too bad, because I'm using Spring MVC anyway and the bagage is already there. But here's the kicker. Because you'll be using this at the front-end facing side of your hexagon, you'll have to pass these datastructures to the other side of your architecture, i.e. through your application layer and domain layer.

And that's where it gets ugly. The last thing I want to do is have a dependency on Spring Data in my application or domain layers. Hell, I don't want to see Spring anywhere near those parts of my application. But if I wanted to do this, I'd have to map these structures to application specific structures to pass them through the system until they hit the infrastructure on the other side, where I'll probably convert them back to `Pageable` instances. Talk about an exercise in futility.

It's something that has been bothering me since I ever laid eyes on something called Spring Data REST (of which I think anyone who's using this library should be lined up against a wall and shot for breaking every sane architectural rule). Some Spring libraries make it really hard to create a proper clean architecture, because they tend to permeate throughout your entire application. I end up abstracting the hell out of some concepts in order to attain some level of framework independence. This is mainly due to the fact that Spring tends to put too much into a single artifact and `spring-data-commons` is a very nice example of that. If they'd extracted the `org.springframework.data.domain` into a separate library, the Thymeleaf extension could've used that instead of the entire library. That package could even exist outside of the Spring ecosystem, as it has 0 dependencies on it (except for it's dependency on `Converter` which should've been kicked to the curb by `java.util.Function` a long time ago). It's something that Spring could learn from when looking at some standardized libraries that tend to split independent parts of their codebase into separate pieces.

So all in all I'll have to ditch the Spring Data extensions for Thymeleaf and yet again write another abstraction to keep Spring out of my inner parts of my application. I'll probably look at the extension's code and probably I'll write an abstraction that can be adapted with a Spring Data implementation (which is how it should have been written in the first place). I'm exalted to use Spring in my infrastructure layers as it makes my life a lot easier, but I just don't want it beyond that. Any library that makes it impossible to do this is forcing me to accept a level of technical debt I'm not willing to accept. 

