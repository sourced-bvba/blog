---
layout: post
category : article
title: "Hello Gluon. Bye Gluon."
comments: true
tags : [technical]
---

Embracing the software crafter mindset, I love to explore new technologies and find new ways to develop software. Creating mobile applications is a part of that and I'm actually enjoying developing small applications.

To simplify things a bit, there are two ways to build mobile applications: native and hybrid. Native applications are developed in specific languages inherent to the platform. In the case of iOS application, that's Swift (or Objective-C if you're into pain) and in the case of Android that's Kotlin (or plain Java if you're into pain). The advantage of native applications is that they can use the full spectrum of functionalities offered by the hardware and you're coding as close as possible to the device so performance is much better than hybrid applications. With hybrid applications, you'll be coding in intermediate languages, such as Javascript. The output in many cases is a web application that is wrapped in web view, so you're basically writing a hybrid native web application. While this is sometimes much faster to develop, the performance is worse than a native application and you're constrained with regards to functionalities: not all native functionalities might be available to you when developing a hybrid application.

When choosing a hybrid application framework, there are a couple of popular ones: Titanium and Ionic seem to attract a lot of people. These are Javascript frameworks and for example Ionic closely relates to Angular, which makes the transition for web developers a whole lot easier. There's also React Native which allows you to build mobile applications using the React framework.

And then there is Gluon.

Gluon is a Java framework that utilizes a VM in order to run Java applications on mobile devices, using JavaFX as the front-end technology. Until recently, it used RoboVM as the VM implementation, but unfortunately that was discontinued. My experiences with Gluon some months ago weren't all that great. It was slow to develop and getting the development environment up and running was a pain in the ass. As a comparison, it took me 3 minutes to have a Hello World application running with Ionic. With Gluon I wasn't even compiling the damn thing yet at that point! However, RoboVM by itself was a pain in the ass to work with, so perhaps it wasn't all Gluon's fault.

I think about 2 years ago, the people at Gluon announced their Gluon VM initiative. They were building a new VM from the ground up, claiming near native performance, and based on the newest Java 9. Rainbows would come from the sky and deliver developers the possibility to build new Java applications on mobile devices with (almost) the same performance as native applications. 

Recently, I saw that Gluon VM finally became a reality, so I revisited Gluon, hoping it would be less of a pain this time to get it to work.

...

...

I started a simple application, single view, one page. A basic Hello World, if you will. Running the application natively went perfecly smooth. And then I tried running the application in the iPhone simulator. 5 minutes passed. 10 minutes passed, 15 minutes passed. It was still compiling. A SINGLE VIEW APPLICATION. Really?! They say you only get one shot at a first impression and Gluon failed miserably here. But perhaps I was doing something wrong. So I went to the Gluon documentation. Zilch. Nothing. Useless. Ok, perhaps Google will help me. It was the Higgs class that was slow as hell to compile my classes so I Google `gluon higgs slow`. For those with a bit of knowledge of quantum physics, you can probably guess how much help Google was. Zilch. Nothing. Useless. Apparently running something in the iPhone simulator is something that is discouraged? I did get something to work on my iPhone 6s however, but it took way to long for comfort. It's just not usable for me at this point and I'd go home frustrated every day if I had to work like this.

Gluon apparently has gained a lot of support from the Devoxx people and the Devoxx apps are developed using Gluon. So apparently it is possible. But it's easy to write Gluon applications if you have a direct link to one of the core developers. Many developers don't have that luxury. And having inadequate documentation in this case is completely unacceptable. And this is something that costs more than double the amount a IntelliJ Ultimate license costs per developer. 

Sorry Gluon, you lost me. Perhaps you are superior to other frameworks like Ionic or Titanium. As a Java developer, I really, really wanted to make this work. But not getting the most basic example to work, and not having any documentation to find out what is going on, I can't see past this. I'll either be developing my apps in Ionic or going completely native if needed, I already know Kotlin and Swift doesn't look that hard. I already got a Swift Hello World running in an iPhone simulator, took me about 3 minutes. Who knew mobile development could be so fast?

*Edit:*

So thanks to a reply on a tweet I got everything to compile (apparently you need to give 4 Gb to Gradle for it to work...) and I got my basic example to run in the simulator. That said, my first impression of the Gluon app was not really that positive, at least in the simulator. Compared to Ionic, where I also have a sample application to play with, and which does way more than my Gluon sample, Gluon is slow. It feels incredibly laggy, especially compared against the snappy response I get when runnning Ionic in the simulator. 

But I have a working example, so I might give Gluon still a go. To be continued.


