---
layout: post
title: "Maven and Ant guys: you’ll never agree. On anything. Period. Deal with it!"
comments: true
categories: [java]
---

Every 3 months or so, you’ll see a new article pop up somewhere written by an Ant (mostly, sometimes Gradle or Rake) lover discussing the horrible nature of Maven and how it works.

People, please! How old are you guys? I’ve used both and I like Maven more. Is that a reason for me to post another rant on Ant? Seriously, most rants are one-way. It’s easy to break down a tool based on some criteria, but how about proposing some solutions to problems some Maven users face that you can easily solve in [your favorite buid tool]? Whoops. Negative criticism is easy, isn’t it?

Recently, I read [this](http://kent.spillner.org/blog/work/2009/11/14/java-build-tools.html) article. It doesn’t take a genius to discover that this blogger is a Maven hater. Or to put it more bluntly: it doesn’t support the way HE wants to build software. Here’s an idea: try and take every feature you hate in Maven and give a Rake or Ant alternative way. Every single one. I’m waiting for the excuses you’ll throw at me.

Let’s see. According to this blogger, it’s really hard to build a war. Last time I built a WAR with Maven, I just needed to change the packaging to WAR. Okay, you do need to know that the WEB-INF folder goes into src/main/webapp. Is that really that hard? The other rant was about the WAR size. Here he actually has a (albeit minor) point. Maven projects (especially frameworks) really need to learn the value of optional dependencies. But then again, I’d like to see Ant projects solve this issue too without their usual solution: provide a README stating which jars should be included when you want to use certain functionalities.

I’ve heard the statement “Why use an entire toolbox (Maven) when you only need a hammer (Ant)”. Okay, you CAN put a screw in a wall with a hammer. The result ain’t pretty, but it works. Does that mean a hammer is the only tool to use. No. Know when to use a tool. Any tool for that matter. Would you throw the screwdriver out of your toolbox because it can’t handle a nail?

Any of the rants can be translated into either incorrect use of Maven or just plain stupidity (or even lazyness): I can point out a lot of Ant, Gradle or Rake misuse that is equally painful. Dependency resolving takes to much time? Use Artifactory or Nexus internally (it’ll even solve some other problems concerning snapshots). Got problems with plugins? Stop omitting the version number in your POM’s (or use a company-wide POM). Too many unused transitive dependencies that should be optional? Contact the author. Got a problem with XML POM’s? Maven 3 has polyglot functionality.
All I’m saying is: stop ranting. Either come up with non-abstract solutions, or shut up. Ranting is easy. Coming up with solutions using your own solutions is less so.