---
layout: post
category : java
title: "Finally an alternative to MySQL's JDBC driver"
tags : [java, database]
---

For those familiar with MySQL and Java it is a well known problem that the JDBC driver for MySQL is released under a GPL license. Off course it is possible to get a commercial license so that you can package MySQL connectivity with your commercial application, but for most of us, more creative solutions were invented. For example, not providing the MySQL JDBC driver with the commercial package and leave it up to the client to get it himself.

Luckily, there is an alternative as of a couple of months ago. MariaDB, a MySQL fork built by the original creator of MySQL, has released a Java JDBC client library that is also compatible with recent MySQL releases. The best part is that it is licensed under a permissive LGPL license, making it a lot easier to package it with your application.

While you should take a look at MariaDB in any case, you can now package MySQL connectivity with your application. You can find the JDBC driver [here](https://downloads.mariadb.org/client-java/). 
