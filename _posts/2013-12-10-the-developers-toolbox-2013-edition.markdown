---
layout: post
title: "The developer's toolbox, 2013 edition"
date: 2013-12-10 06:33
comments: true
categories: java
---

Another year has almost come to an end and another year of technological advances has passed. I'd like to share my toolbox going into 2014. <!--more-->

### Build system: [Gradle](http://www.gradle.org)

I've used Maven professionally for the last 6 years or so and I don't regret for a minute choosing Maven as my build platform. But recently I've made the switch to Gradle. The choice is purely personal, as I still believe both are functionally equivalent (there are still things that Maven can do that Gradle can't, and vice versa). But most of the open-source projects I use on a day-by-day basis have switched to Gradle (Spring, Hibernate or other JBoss OSS products), which means I need to learn it anyways. That said, I personally feel I'm a wee bit more productive with Gradle. I can do the same builds with Maven, but writing the POM for the same project might take me 10 minutes longer. The things I really, really like about Gradle are that my build scripts are now a lot more readable, centralized (I only have 1 gradle script for a 10 module project) and extensible. Especially that last item is important, at least to me: I favor composition over inheritance and Maven just doesn't allow for flexible composition of a build script like Gradle does. So as far as building my projects goes, I'm going full Gradle.

### Version control: [git](http://www.gitscm.org)

At the moment, git seems like the logical choice. I've tried other DVCS systems such as Mercurial or Bazaar, but I keep falling back to git. Sure, the command line can be a pain in the ass (Mercurial is a lot more sane in that area), but the git flow addition alone is enough reason for me to stick with Git (I know, there's hgflow, but it's not stable yet). I can live with the fact that I can seriously shoot myself in the foot with git when mucking around with git commit logs (but even then one could debate it's deliberate).

### Text editing: [Sublime Text 2](http://www.sublimetext.com/)

My revelation of 2014. I've been using Notepad++ for god knows how long and I was quite happy with it, until I stumbled upon Sublime Text 2. It does everything I did with Notepad++ and a lot more. It's free to use (actually, it's a indefinite trial), although I'll be buying a license in the near future. If you haven't tried it before, take a look. 

### IDE: [IntelliJ IDEA](http://www.jetbrains.com/idea/)

Jetbrains just released version 13 and frankly I'm stunned with this latest incarnation of their flagship IDE. IntelliJ 12 was already my IDE of choice, but 13 really adds a lot of features (some of which I didn't even know I wanted). Among other they include Search Everywhere, Presentation Mode (we'll be seeing a lot of that in the upcoming conferences I bet) and enhanced Gradle support. Add support for the latest frameworks (Spring, CDI, JSF, ...) and you have a winner. There's no way I'll be switching to Eclipse or (shiver...) NetBeans ever again and I happily pay IntelliJ's cost.

### OS: [Ubuntu Linux](http://www.ubuntu.org)

I've gone through more Linux distributions this year than I can remember. Mint, CentOS, Debian, Fedora, all of them passed during my search of the ultimate development distribution. In the end, I ended up with the one I started off with: Ubuntu. I don't even remember why I left Ubuntu in the first place (I think it had something to do with Unity getting in my way), but eventually I accepted the things I didn't really like. 
Yes, I've tried Windows 8. For 2 full days. Because it came pre-installed on my new PC. Maybe it's me, but I have developing Java applications on Windows. I use the command line a lot, and Windows' native command line just plain stinks. I could install Cygwin, but if you've gone that far, you should use Linux anyway IMHO.

### Command line tool: [Terminator](https://apps.ubuntu.com/cat/applications/precise/terminator/)

While on the command line subject, another discovery this year. I've used Ubuntu's built in terminal application, just because it's easy. But then I got a tip that I should try Terminator. After trying it out, I never switched back. Why? Terminator is incredibly handy when you have 2 or more terminals open. The ability to have split screen terminal windows in a layout that I choose is a winner feature to me. It's the supercharged version of Ubuntu's built-in terminal client.

### Java frameworks: anything OSS

I'm not really that picky anymore on frameworks. If it does the job well, I'm using it. For ORM purposes, I mainly use Hibernate, but if needed I'm happy to use EclipseLink or OpenJPA. If I need a CI container, I'll probably use Spring, but I've used Weld as well (however I must say I really prefer using Spring). The point is: I need it to be open source. I need to be able to debug if things go awry. Having been forced to use closed source IBM products (which allegedly support Java standards), I can't really stress the importance of open source.
I could give a entire list of frameworks I have in my toolbox, but that would make this post explode. 
The same goes for application servers. I don't care whether it's JBoss AS (or WildFly), Tomcat or Glassfish. But given the choice, I'll stay as far away as possible from WebSphere or WebLogic. They support all the standards, but for some reason something always works differently on those platforms.

### JVM Language: [Groovy](http://groovy-lang.org/)

I've tried a lot of JVM languages this year: Scala, JRuby, Ceylon, Kotlin and the new Groovy version. As far as practicality and usability is concerned, Ceylon and Kotlin still aren't up to the level I need them to be in order to consider them a feasible choice for projects. JRuby was on my list as I use a lot of Ruby tools during development (Redmine for example), but there's someting about the Ruby language that puts me off. I'm sure it's a personal thing. Scala... well I tried, I really did. But I just can't get used to the syntax. Perhaps in a limited subset of projects I might consider Scala, but not for big projects. Java code can be difficult to read but Scala code can be impossible to read. Groovy seems to hit the sweet spot. It's close enough to Java to be familiar enough to start off quickly, but it adds so much useful features (closures, BigDecimal based math, GString, properties, ...). My favorite testing framework, Spock, is also written in Groovy

### Unit testing: [Spock](http://code.google.com/p/spock/)

As far as testing goes, I now try to use Spock as much as possible. I'm a big fan of TDD and BDD, and Spock seems to be the solution that has the smallest learning curve. The built-in mocking features are excellent and the assertion syntax feels natural. It integrates perfectly with other framework such as Spring and supports Hamcrest matchers.

### Web applications: [AngularJS](http://angularjs.org/) and REST

Having done some work with Vaadin and GWT, I'm now convinced the most effective web applications are done through with HTML5 and a client-side framework such as AngularJS. It does have somewhat of a learning curve, but it offers so much more flexibility. With client-side frameworks and REST, you can completely decouple your front-end application from your back-end. If you have a front-end model and a CQRS-based backend, there's nothing keeping you from independently developing you front-end as long as the back-end is capable of providing the data required. This aspect comes in handy in the early stages of a project when you're still prototyping.
Also, with the front-end completely on the client side your designers have complete freedom, which means you can provide them with the flexibility needed to support different platforms.

### Architecture: CQRS 

Having programmed countless applications the 'traditional' way in a layered architecture, I've come to believe this is not the right way to write applications anymore. Unless you're certain your application will not have a large user base you should be used a CQRS based architecture. Which means you should design your application in a way that write operations can be easily separated from read operations. CQRS involves a lot more than that, but already having that kind of separation will make your life a lot easier when you run into serious performance issues. If you have the means to have a architecture entirely based on CQRS principles, certainly go for it. Just [don't go overboard](2013-12-01-on-eventually-consistent-data.md).

### SQL Storage: [PostgreSQL](http://wwW.postgresql.org)/[MariaDB](http://www.mariadb.org)

I'm still in doubt which one would the preferred choice. PostgreSQL seems an easier sell to management due to its more 'enterprisey' reputation and the fact it very similar to Oracle (which is in fact the only non-OSS database I would recommend). MariaDB (not MySQL) is however a lot more easier to use and isn't as complex as PostgreSQL. Feature-wise, both are equivalent to me. In highly-concurrent systems, you'll start to see the difference, but in that case I wouldn't use a SQL database anyway (at least, not anymore).

### NoSQL storage: (still) [MongoDB](http://www.mongodb.org) 

Using MongoDB is a obvious choice to me: it's currently one of the most (if not the most) popular NoSQL database out there. It has great support and does what it is supposed to do. It's easy to install, simple to use and has excellent performance. 
But I also have to say I've been looking into other NoSQL datastores lately, so it could be that my choice changes in 2012. Neo4J looks really interesting, as do ElasticSearch and CouchDB. More to come in 2014. 

### Documentation: [AsciiDoctor](http://asciidoctor.org)

Writing documentation is something no developer loves. Since I've been using AsciiDoctor to write documentation, I don't really dread writing documentation that much anymore. The fact alone that I don't need to start up a word processor to write my documentation (if there is something I loathe more than documentation, it's Word) is a definite win for me. The documentation can be generated in both HTML(5) and PDF format so that you could provide and offline and online version of your documentation.

Tools that are also worth mentioning are [Liquibase](http://www.liquibase.org), [Mockito](http://code.google.com/p/mockito/) and [Lombok](http://projectlombok.org/).

If you have other tools that you have in your toolbox and which you couldn't live without, please share in the comments.
