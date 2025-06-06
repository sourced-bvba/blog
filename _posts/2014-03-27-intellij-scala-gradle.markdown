---
layout: post
category : announcements
title: "IntelliJ, Scala and Gradle: revisiting hell"
tags : [gradle]
---

So I finally made the decision on trying to learn Scala. Little did I know I was in for another round of IntelliJ integration hell. Let me rephrase that: IntelliJ with Gradle hell.

I love Gradle. I love IntelliJ. However, the combination of the two is sometimes enough to drive me utterly crazy. Now take for example the Scala integration. I made the most simple Gradle build possible that compiles a standard Hello World application.<!--more--> 

``` groovy
apply plugin: 'scala'
apply plugin: 'idea'

repositories{
    mavenCentral()
    mavenLocal()
}

dependencies{
    compile 'org.slf4j:slf4j-api:1.7.5'
    compile "org.scala-scala-library:2.10.4"
    compile "org.scala-scala-compiler:2.10.4"
    testCompile "junit:junit:4.11"
}

task run(type: JavaExec, dependsOn: classes) {
    main = 'Main'
    classpath sourceSets.main.runtimeClasspath
    classpath configurations.runtime
}
```

First I stumbled upon the first issue: the Scala gradle plugin is incompatible with Java 8. Not a big issue, but this meant changing my java environment for this build, so it is a nuisance. Once this was fixed, the Gradle build succeeded and Hello World was printed out. 

I opened up IntelliJ and made sure the Scala plugin was installed. Then I imported the project using the Gradle build file. Everything looked okay, IntelliJ recognized the Scala source folder and provided the correct editor for the Scala source file. Then I tried to run the Main class. This resulted in a NoClassDefFoundException. IntelliJ didn't want to compile my source classes. So I started digging.

Apparently, the project was lacking a Scala facet. I'd expected IntelliJ to automatically add this once it saw I was using the scala plugin but it didn't. So I tried manually adding the facet and there I got stuck. See, the facet requires you to state which scala compiler library you want to use. Luckily IntelliJ correctly added the jars to the classpath, so I was able to choose the correct jar. This, however, did not fix the issue as IntelliJ now complained it could not locate the scala runtime library (scala-library*.jar). This library was however included in the build. If you were to choose the runtime library as the scala library, it would complain it cannot find the compiler library. And this is where I am now: deadlocked. 

There is an issue in the bugtracker of IntelliJ [here](http://youtrack.jetbrains.com/issue/SCL-4704) but it's been eerily quiet at Jetbrains on this issue. As it is, it's impossible to use IntelliJ with Gradle and Scala unless you're willing to execute every bit of code including unit tests with Gradle instead of the IDE (which in effect defeats the purpose of an IDE). And I'll die before adopting yet another build framework (SBT) that is supposed to work.

Honestly, I really don't know whether I want to learn Scala anymore. Just the fact that you can't compile Scala in the most popular IDE at the moment when using the most popular build tool at the moment is something I cannot comprehend. Forcing me to adopt a Scala-specific build tool is unacceptable to me. 

If I were TypeSafe, I'd put an engineer on this and fix this as this would seriously aid in promoting the language. If it were easy to adopt Scala in an existing build cycle, it would pop up on more radars than it would right now.

But it's not just Scala and IntelliJ: most newer JVM languages struggle with IntelliJ. This is a real pity as this either forces me to change my IDE (i.e. Ceylon has its own IDE based on Eclipse) or not consider the language. As it is, the current viable option with IntelliJ is Java and Groovy (and Kotlin, but it's not even near production ready quality). Wouldn't it be nice to only need one IDE for all development? I couldn't care less if it would cost $500, I just want things to work. I'd love to be able to write my AngularJS front-end that's consuming my Scala/Java hydrid backend reading data from a MongoDB that's feeded data from my Arduino sensors (for which I've written and uploaded the sketch from that same IDE).
