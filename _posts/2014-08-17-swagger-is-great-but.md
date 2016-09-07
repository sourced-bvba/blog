---
layout: post
category : article
title: "Swagger is great, but..."
comments: true
tags : [java]
---

Nobody likes to write documentation. However, in the REST age where REST-based webservices are obiquitous, documentation for public webservices is a necessity. There are a lot of tools out there that provide REST documentation generation, but there's one that stands above the rest (pun intended): Swagger. Swagger is a JSON format that describes a REST webservice. However, the strength of Swagger is the UI, which is a JavaScript HTML5 UI that is just awesome.<!--more-->

Even better, the integration with Spring MVC is top notch. Configuring Swagger in a Spring Boot application is as simple as adding a dependency and adding an annotation (hint for the Swagger guys: provide an autoconfiguration for Spring Boot).

That being said, there are some things that really, really annoy me. See, Swagger is built using Scala. Which also means that if you want to use Swagger, you'll be pulling the entire Scala runtime into your build. For a Spring Boot application just serving a basic Spring MVC REST service, this means the build goes from 15 Mb to 48 Mb. Urgh. Tripling my build size just to add documentation is enough for me to call it a day and throw it out of my build. 

Another thing is that Swagger really doesn't work in a microservices environment, at least not with Spring MVC. Swagger needs to be in the build with the REST controller, so this means you'll be including Swagger into each and every microservice deployment (dragging the 33 Mb overhead everywhere you go) and you'll have multiple Swagger JSON files. Double urgh.  I have yet to find an aggregator that merges different Swagger JSON's into a single one.

Of the 2 issues I have with Swagger, the size issue is the one I find the most annoying. I understand that it's cool to use Scala but if using a library would mean my build grows 3 times in size I would think twice of using it, especially if it's just for generating documentation. I personally think Swagger should be lean and mean. Just cutting Scala from the library would reduce the overhead by 20 Mb (I use Groovy rigorously and even then I don't like the fact that it adds between 4 and 8 Mb to my build). But it's also just POM sloppiness: pulling in mockito (should be a test library), 2 versions of cglib, JodaTime libraries (which for Java 8 users might not be desirable), ... Some proper POM maintenance seems to be in order.

And as for the JSON aggregation, I don't think it would take me too long to write something in JSON that reads the different JSON's and aggregates them into a single one (which would also solve the nagging CORS issues you'd be having otherwise). Come to think of it, it would be great to make an aggregator to which applications can push their Swagger JSONs on startup. That way you would have some sort of automatic aggregated documentation that updates itself after each deployment and to which adding a microservice would be a breeze. Documentation as a service, I like it. Another day, another post, another idea.