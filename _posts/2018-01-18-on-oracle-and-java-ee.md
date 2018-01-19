---
layout: post
category : article
title: "On Oracle and Java EE"
comments: true
tags : [opinion]
---

First of all, let me start by saying I am by no means a Java EE advocate. Not even by a long shot. I've been working with Spring for the last 10 years or so and aside from a couple of consultancy assignment where I was basically forced to work with the EE ecosystem (I'm looking at you, Websphere), Spring has been my weapon of choice when it comes to enterprise development.

Last week, the Java EE Guardians published an open letter addressed to Oracle regarding the rebranding of Java EE and the consequences of such a rebranding. Perhaps a little history: Oracle open sourced the Java EE codebase, which then moved to the Eclipse Foundation. With that move, a rename was pushed trough, Java EE was now called EE4J. You see, Oracle still retains ownership of the Java brand and therefor didn't allow continued use of the Java EE brand in it's current name. Additionally, most EE code extensively uses the `javax.*` packages, but Oracle also retains ownership of the use of those packages. This means any new API's coming out of the EE4J project would not be allowed to use the `javax` packages. 

In their plea, the Guardians asked to Oracle to retain the possibility to keep on using the Java EE name and to be able to use the `javax` and `javax.enterprise` package names for new APIs, under licence if it must. Oracle's answer was clear: no. Java is a protected trademark for Oracle and they're not willing to let it go. 

Now, what's all the fuss about, you may ask? A package rename is for most a single line `sed` script, and the name of a product can change, AngularJS changed to Angular after all.

Well, Java EE (or EE4J) is considered as being the 'standard' approach to building enterprise Java applications. It's API are considered standardized through the input of various enterprise stakeholders by using the now defunkt JCP. On the other hand you had Spring, which was a product of Interface21/SpringSource/Pivotal, which is now a very popular, if not the most popular, appraoch to building Java applications. However, it's not considered a standard, at least not by the Java EE guys, as they laid claim to that title. 

Being considered as a industry standard has many consequences. Some companies only wish to work with standard because of the perceived safety using a standard means. Integrators tend to focus on integrating with a standard instead of a proprietary approach. So if Java EE loses the perception of being the official standard, it has a lot to lose. 

Personally, I never considered Java EE the standard for enterprise development. It was a standard, sure, but on the same level as Spring. People either chose Java EE or Spring, not unlike one would choose Makita or Milwaukee when choosing an angle grinder. For me, the moment Oracle relinquished control of Java EE and turned it over to the Eclipse Foundation, it became a former standard and thus now proprietary apporach, just like most other Eclipse projects. It just officially became 'the framework formerly known as Java EE' and just one of the ways to build an enterprise application.

So if EE4J is forced to use it's own non-standard packages and no longer has a name that was linked in our collective mind to a standard, what makes it different from Spring. 

Nothing.

But Spring cannot exist without Java EE, Spring implements a lot of Java EE standards, you might say. Sure, but that was out of necessity and because they could be integrated with relative ease. If they didn't exist, Spring would have found a way to do it themselves. And they have done it themselves: there are a lot of things within Spring that do not have a Java EE equivalent.

So Java EE has a lot to lose because of Oracle's standpoint. There's a world of difference in perception here between a Java EE standard and an EE4J API. This is probably why the Java EE Guardians are fighting so hard to retain the name and the packages. They know it's a war on perception, a war that in my eyes they have been losing for some time. And Oracle's latest move could prove to be the nail in the coffin of Java EE. Without the protection of being considered 'the standard', they're losing one of their main selling points: that they, in fact, ARE the standard. 

Personally, I'd rather see the EE4J project just become a place where the old Java EE get a place outside of Oracle's control. These are very stable APIs and aren't that prone to change anyway. And I'd rather see the brains and knowhow of people like Reza Rahman or Adam Bien being put to use to help drive new features in the more fast-moving framework that Spring is. 

Perhaps it's time to bury the rivalry and rally under the same flag. Just imagine what could be achieved by joining forces. Spring would become the de facto standard (if it isn't already), bringing everything and more to the table of current Java EE developers. Spring would need to provide a bit of support to accomodate Java EE developers, building some bridges to make sure Java EE developers need to change as little code as possible. Like I said, Spring supports just about every standard that Java EE has, with some small exceptions. 

To me, it's clear that Oracle doesn't give a damn about the future of the EE4J project, they're just protecting their copyright. And that's their right to do so, they own the name after all. But to me, they're killing Java EE in the process. Or at the very least the perception that Java EE or EE4J is the standard, which may be equally damaging. 

Lastly, I don't get why the Java EE Guardians keep on investing so much time and energy in a battle that seems less and less likely to have a good outcome. I respect their efforts, but I just don't get it. Instead of antagonizing each other, perhaps it's time that both sides joined forces and focussed on what really matters for enterprise Java developers: providing solutions for the problems we're facing today. We've had two competing standards for over 10 years now. Perhaps the time has come to choose. Giving Oracle the bird on the way out is just an added bonus. God knows they wasted enough of our time with these shenanigans.

