---
layout: post
title: "Book review: Gradle In Action (Manning, 2014)"
comments: true
categories: [book review]
---

As a MEAP subscriber I get the privilege of reviewing some of the new books released by Manning. Some time ago I received Gradle In Action and my review is a bit overdue, so here it is. I've been a Gradle enthousiast for the last couple of years, migrating from years of working with Maven, so I was very keen on reading this book.<!--more-->

The aim of the book is to both introduce newcomers to Gradle and provide in-depth information for those already accustomed to Gradle. While the book mainly uses Java technologies to illustrate the possibilities of Gradle, most items are also applicable to other technology stacks as well. There is a nice chapter on polyglot programming and even how you can integrate things like Grunt into your automatic build.

The first three chapters are mainly introductory and cover Gradle installation and provide some background on why Gradle is a great choice for building projects and why build automation is an important part of development. Command line usage is covered nicely, which I think is important as Gradle is at its most powerful when used from the command line. The introduction is ended by showing you how to set up a sample Java web project and provide you with a lot of ready-to-reuse code snippets. If you're already familiar with Gradle, you can race through these chapters. If you're already using other tools like Maven or Ant, chapter 9 is very useful.

The next couple of chapters go into the gradle build file. Gradle is built around the concept of tasks and the book does a nice job in explaining how tasks work and how easy it is to write custom tasks. Dependency managemement is covered in depth and gives enough information for both Ant+Ivy and Maven users to quickly formulate their dependencies in gradle build files. It's also nice that they cover how you can handle version conflicts and how you can manipulate the dependency resolution mechanism to override or enforce specific versions.

Most of us work with multi-module projects and the book does a nice job in covering the specific issues you'll encounter with such projects: intermodule dependencies, shared configuration between module. The book also shows the pros and cons between a single build file and build files per module (but clearly show a bias towards splitting them up). Off course testing is covered also in this book using JUnit as a reference (an explanation on how to plug in other testing frameworks is provided though). The three levels of testing, being unit, integration and functional, are explained in a clear and concise manner.

The last remaining chapters really go into detail on some of the features in Gradle you won't probably use every day, ranging from Gradle plugins, artifact publishing, integrating code analysis tools such as Sonar to instructions on how to build a deployment pipeline. The part on Gradle plugins could have been a bit more elaborate, but that would have probably taken the book way out of the scope that it was intended for. Perhaps a Gradle Plugins In Action would be more suited for that kind of detail (hint Manning).

All in all, I highly recommend this book for everyone starting to use Gradle. For experienced Gradle users most of the items in the book would already be known, but there are still a few gems in the book worth buying it (like the Grunt integration or integration/functional testing part). It's well written and I consider it the most complete Gradle book to date.

You can buy the book at [here](http://www.amazon.com/Gradle-Action-Benjamin-Muschko/dp/1617291307/ref=sr_1_1?ie=UTF8&qid=1393496383&sr=8-1) at Amazon starting March 3.
