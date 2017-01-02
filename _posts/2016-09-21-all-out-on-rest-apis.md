---
layout: post
category : article
title: "Why I'm putting all my cards on hypermedia APIs at the moment"
comments: true
tags : [java]
---

A couple of months ago I switched jobs and went from a product-centric, sales
driven environment to a project-centric, tech-driven environment. Needless to say,
this was a bit of an adjustment for me, despite the fact I did project-centric
work for over 6 years.

My new job required me to become a full-stack developer. I've always been a backend
developer, rarely having touched a web frontend. Over the last 4 months, I've learned
more on Javascript frameworks than I have the last 10 years. And that's a good thing.
But it also made me realize how important a good backend really is.

You see, most frontends require a well thought-through REST API in order to be able
to deliver functionality quickly. And just like I've learned a lot about things like
AngularJS, I've learned how good RESTful API actually look like. This also was a
big eye-opener for me. Most of us see concepts like HATEOAS as a fine addition on
good RESTful APIs. Even I thought this was something that was separate from
developing an actual RESTful API. However, I was wrong. A concept like HATEOAS is at the
core of a good RESTful API and should be a prime concern whenever one starts to develop
such an API.

There are many ways to implement a self-discovering API using REST. There's HAL, JSON-LD,
Collection+JSON and SIREN. Professionally I'm now using SIREN and after experimenting with
the other approaches I can see why my company has chosen it. It's easy, feature-packed and
quick to implement in Java (which is a big plus for me).

But a tool is just a tool and it's still easy to create awful RESTful APIs using SIREN. I've
made this mistake in the past. RESTful APIs need to have clear boundaries and need to be given
great thought. Most of us start developing it and make it grow organically as the need arises.
This is the wrong approach. Like in DDD, you need to know what your domain looks like and model
your APIs accordingly. In brown-field development, a REST API is something that is created as an
add-on and tends to choose the path of least resistance, especially if the originating code it uses
does not adhere to DDD standards (like mixed bounded contexts). The result is a mess and I'm not proud
to say I've made my share of bad RESTful APIs because I didn't adhere to that prerequisite.

This is why I'm currently devoting most of my time researching how to create good RESTful APIs. It
also means I'm brushing up on my DDD knowledge, because I see the two seriously intertwined. I've
come to the point that I realize that when you develop a RESTful API, you shouldn't even look at the
existing system. You should look at the domain and work from there, identifying the links and actions
needed to navigate the domain. Making a RESTful API built with a good domain talk to an existing
backend is easy. Fixing a bad RESTful API because you worked bottom-up is not.
