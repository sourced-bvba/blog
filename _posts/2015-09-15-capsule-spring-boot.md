---
layout: post
category : article
title: "Small Spring Boot jars with Capsule"
comments: true
tags : [java]
---

Those familiar with Spring Boot already know that Spring Boot applications can result in quite large jar or war files. In most cases, this is a not an issue, but often between versions you're essentially only touching maybe 5% of the entire size of the jars, while the other 95% consist of dependencies which haven't changed. This is the concept of a fat jar. However, with the 1.0.0 release of [Capsule](http://www.capsule.io), this might change.<!--more-->

## Capsule

Capsule is a packaging system for JVM applications. Just like Spring Boot, it enables you to build fat jars. However, there are a lot of things Capsule is capable of doing. One of the most interesting aspects is that it can build quite small jars which download their dependencies on demand, cache those dependencies and reuse them if possible. If you know Java Webstart, this soundly very familiar. 
This means a Spring Boot jar or war file can be reduced from tens of megabytes to a couple. The downside is that the initial startup does take a while because it needs to download the dependencies from the internet and therefor you also need an active internet connection. However, all next startups will be very fast and will only download any changed dependencies.

## Gradle

As Capsule essentially does the same thing as the Spring Boot repackaging process, you can replace the boot plugin in your Gradle build file with the [capsule plugin](https://github.com/danthegoodman/gradle-capsule-plugin). You need to add the following to your build file.

``` groovy
plugins {
    id "us.kirchmeier.capsule" version "1.0.0"
}
```

Then you can choose whether you want a fat jar or a slim jar that download dependencies on demand. The fat jar is by far the easiest and behaves the same as the standard boot plugin (I even noticed fat Capsule jars are a bit smaller for some reason). You need to add the following task to your build file.

``` groovy
task fatCapsule(type: FatCapsule) {
    applicationClass 'com.example.YourApplication'
}
```

Then a simple `gradle clean build fatCapsule` does the trick.

To build a jar that downloads its dependencies on demand, you need to have a different task:

``` groovy
task mavenCapsule(type: MavenCapsule){
    applicationClass 'sample.RsApplication'
}
```

The command to build this jar in this case is `gradle clean build mavenCapsule`.

But there is so much more you can configure with Capsule. It has the concept of caplets, that aside from the MavenCaplet which downloads dependencies on demand,  also has implementations for native packages for GUI applications, support for daemon applications, support for running in a minimal container or in a sandboxed environment. You can configure JVM parameters that Capsule should use to start your application (such as -Xmx settings) and even define which JVM Capsule should use, detailed to the level of the update version. It can set the boot classpath and define java agents (which can even be defined by their Maven coordinates).

## Conclusion

For me, with the discovery of Capsule, there is no reason whatsoever to use the Spring Boot repackaging functionality. Capsule provides everything that the repackaging does AND much, much more. It enables you to create much smaller jars and tweak default start behavior. As far as I am concerned, I'm not using the Spring Boot plugin in Gradle anymore.
