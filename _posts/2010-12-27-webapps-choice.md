---
layout: post
category : "opinion"
title: "Why I think web applications aren’t always the best choice"
tags : [swing]
---

Web clients are hot. Hip. Sometimes even hype. But having worked on different projects, both fat clients and web applications, I’m starting to grow a consciousness on when a web application is appropriate and when the choice for a fat client would be a more sane one.

Fat clients have a bad reputation. Deployment issues being the main culprit here. With web applications, you just deploy it on a server and every user has instant access to the application. Fat clients, for example using Java Webstart, need to be installed on the local client. They also require a JVM. Updating is not instantly, but requires a restart in most cases (unless for example you’re using some fancy modular architecture that allows updating on the fly). From an operations standpoint, fat clients are maintenance burden that isn’t popular. But Java Webstart has evolved, and in the light of that evolution, perhaps it is time to re-evaluate our behavior on defaulting to a web client.<!--more-->

Why would one choose to develop a fat client? Here are my reasons:

**1. Anything the client wants**

Javascript libraries such as JQuery, Prototype and Dojo made the dream of a rich web client a reality. But the developer is still bound by the same restrictions as before: the web browser. Although HTML5 is bringing renewed life to the rich web client, there are still a lot of things a web client just can’t do. And if you get something to work, chances are some browser in the wild will behave differently and you’re off building yet another workaround for IE6, Mozilla or Chrome. Off course, you have Flex, but I have yet to see a enterprise-size Flex application that is delivered faster than a Swing application.
With Java Swing, the sky is the limit. You can do just about anything with a Swing application. And it works every time. Although really rich Swing applications require a thorough knowledge of graphics, animation and Java Swing details, there almost nothing a client can ask that you can’t deliver. And as most of us know, clients can be very demanding. Who of us haven’t gone off for a search for that one AJAX component the client really wants?

**2. Offline usage**

Web clients are only useful as long as the client has a network connection. Sometimes, this is not possible, for example representatives on the road. Okay, there is 3G, but what if your representative happens to be in an area that doesn’t have coverage?

**3. Only data**

Swing applications, once loaded on the client, only need raw data to perform their actions. No presentation state is ever transferred to the client, resulting in lower bandwidth usage. AJAX does mitigate this problem a bit for web applications, but you’re still hauling more bits than needed.

**4. Power users need powerful UI’s that don’t require a mouse**

Data entry users hate web interfaces. It’s a fact. Most such users are used to terminal screens, only needing the keyboard to operate the application. Writing a keyboard only web application is not an easy task. Most power users really don’t care whether their application can be changed without restarting (one of the main selling points of web applications), they just want functionality that mimics their earlier terminal screens. For Swing developers, building a keyboard only application is quite easy. You have complete access to the key handling system of the underlying OS and as such you can do just about anything.

To me, web applications cause too many compromises. Most web developers are very much aware of the limitations of the web browser and the web application frameworks’ presentation capabilities. And so they (including me sometimes) try to convince the client to settle for less. And we really have to convince them, as the alternative is days of web development hell. Power users are used to the nice functionality of their Office suite, ERP and other rich client applications. And they don’t understand why it’s so hard to do the same thing in the browser. In the end, they give up and settle for less. As we do.

So when do I think web applications are appropriate? Consultation and reporting. The web is ideal for data visualization. Being able to pull up data fast and efficient is what the web is great at. But stop making data entry web applications. Your users really don’t care whether there data entry screen is in a browser or in a separate application. They only want it to be fast and more importantly, they don’t want to use the mouse.

**Addendum**

Today, I had to work on some issues on a web based data entry and visualization tool. All data originates from a mainframe. In trying to solve my issues, a colleague of mine helped solving some really hard issues on the web client. For verifying the data I just entered, he opened up the terminal screen and started an environment a lot of power users in the real world are accustomed with: a console-based mainframe application. I was astonished by the speed and ease of use of such a system. Sure, it didn’t have a stunning UI, but 1) it was very functional, 2) didn’t require the mouse, 3) extremely fast and 4) ran anywhere, as long as you had a terminal emulator. This is the kind of applications users like to see from a functional point of view. I really can’t imagine how one would attempt to create the same level of functionality, in all it’s aspects, with a current day web application. Somewhere along the web way, function gave way to form and unfortunately, nobody wants to build a plain looking web application anymore. A lot of desktop/mainframe-to-web migrations go awry because of this, power users just lose too much.