---
layout: post
title: "Book review: Release It! (Pragmatic Bookshelf, 2007)"
comments: true
categories: [book review]
---

So here's a review on a book I've been planning on reviewing for a long, long time. You may be wondering, how relevant can an IT book be if it's more than 6 years old? Well, this book still is and will be for a very long time. I read it in just a couple of days, although I have to admit it was planned: I slipped on vacation in the Dominican Republic and had to have my ankle put in a cast. So the next week it was me by the pool with a couple of books :).<!--more-->

## What is it about

Writing software is not that difficult, it's what happens after you write the software. Getting software succesfully into production and keeping it running is a lot harder. Most software lifecycles go like: write, deploy, bugfix, deploy, survive crash, bugfix, pray to the gods, deploy and so on. This book is about how you can prepare for having your software deployed for production and how you can build in mitigating solutions for some of the most common problems you'll encounter in production.

The book is divided into four parts:

* **Stability** or how to keep your software going when thing go wrong
* **Capacity** or how to keep your software survive unexpected loads 
* **General design** or how you can solve security and physical network problems
* **Operations** or how to keep your support and operation colleagues happy
 
The book shows the anti-patterns that will occur in your applications and the patterns that can mitigate the issues caused by those anti-patterns

## Stability

Most software isn't a island on its own. It's dependent on a lot of different systems and being prepared for a failure of one of those systems is what this part is all about. Most stability anti-patterns that are mentioned are the ones that a seasoned developer knows such as slow responses, cascading failures, integration points and off course users. However, some of them are a bit more complex. One of the anti-patterns mentioned for example is SLA inversion. Unmitigated, it basically comes down to the fact that the maximum SLA your application can provide is the lowest SLA of your dependent systems. So if you have a dependent system without an SLA, you cannot provide one either. Another one you probably won't think about while developing: how does the system react when there is suddenly an unexpected, unprecedented load on the system? The example they give is an error in advertising that prices an articles for 1 dollar instead of 100 dollars. You'll have a rush on your system like nothing you've seen before. Odds are your system will go down due to the massive load. Same thing happens when you site gets Slashdotted.
Naturally there are a lot of patterns that help mitigating these issues. Safe to say, it's never a bad idea to incorporate as many of these patterns as you can, but some of the more important ones that are discussed are building in timeouts, failing fast (and using a circuit breaker to ensure integration points with problems are failing fast), ensuring a steady state at all times and using bulkheads to stop cascading failures.

## Capacity

When developing software, you have a idea on how many users, concurrent or else, you can expect on a production system. However, this part of the book shows you what can happen if you make the wrong assessment and what will happen if you do. It also shows you what dangers things like the reload button and AJAX can bring. 
One of the things that you'll probably already have encountered is database pool saturation. This happens when you have underestimated your concurrent usage or when you're using connections in long-running transactions. Another one of the problems you probably have seen before is issues with AJAX. When overusing AJAX, you'll be hammering your webserver with requests. This, in combination with using AJAX to not only transmit data, but also presentation (sending HTML over AJAX) can bring down a webserver real quick.
There aren't a lot of patterns to cope with these issues. Most of them are already in use in production systems, such as connection pooling, so it seems to me the goal of this part is to make sure you don't implement the anti-patterns.

## General design issues

A lot of developers develop software on their own PC, on a single application server without any clustering or failover. Most of the time, they're using a super user in order to develop as quickly as possible. 
This part is about making sure your software is prepared for load balancing, clustering and how to make your application as secure as possible. It also stresses the importance that a QA environment should perfectly match a production environment. You don't want to be solving issues on production that can't be reproduced anywhere else than the production environment.
One of the things they also mention is adding administrative interfaces to your system. When everything goes wrong, the command line is your friend and having an administrative command-line interface will make you and the persons supporting your software a lot happier.

## Operations

So the system is in production. Things go wrong, despite implementing a lot of the patterns in the book. This part is all about how to add features to your software that help you to quickly identify the problems. For example, it stresses the usability of logging. Having a readable, uncluttered log file is crucial in order to identify issues. Another important feature is live monitoring. In the Java world, we have technologies like JMX and tools like JaMon and JavaMelody. In operations, you have monitoring systems such as Nagios. Not having things like this in production is just asking for problems. 
Last, but definitively not least, the importance of releasing software fast and painless is emphasized. If things go wrong you want to be able to deploy a solution as quickly as possible and not have to go through a release process that take countless hours or even days.

## Conclusion

This is one of the most interesting books I've read so far. It's up there with the Mythical Man Month and Effective Java. It's a must read for every developer as it will make him or her a better developer. It will also help that developer to sleep a lot sounder when the production date for their software comes closer. 
But not only developers should read this book. IT department heads can also learn a great deal from this book. Most of the time, they are focussed on adding business value to the system. Well, the lesson they should learn from reading this book is that it pays off to make time for implementing technical features such as the ones described in the book. The last thing they want is a fantastic piece of software that goes tits up at the first sign of trouble.
All in all, it's a must read!

You can buy the book at [here](http://www.amazon.co.uk/Release-It-Production-Ready-Pragmatic-Programmers/dp/0978739213/ref=sr_1_1?ie=UTF8&qid=1373812004&sr=8-1&keywords=release+it) at Amazon.
