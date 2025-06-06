---
layout: post
category : opinion
title: "SQL, NoSQL, I don't care. It's about events!"
comments: true
tags : [java]
---

The NoSQL/SQL debate has entered a new chapter, as articles seem to be popping up all over the place
where NoSQL is being talking down in favor of a renewed trust in SQL (or more correctly ACID compliant)
system. For example, Google is looking to its F1 database in favor of standard NoSQL solutions.

But is this discussion actually important? Sure, it's data, but that's all it is. Frankly, I don't care 
whether data is stored in a SQL or NoSQL database. It could be stored on punch cards for all I matter,
just as long as the system is reactive.<!--more-->

I'm a huge fan of reactive programming and CQRS/event-sourcing architectures. And to me the choice between an
event-driven system or standard transactional, synchronous system is far more important than the persistence
of the data within such a system.

A system handling data can be seen as a system merely handling events: adding, removing, updating data are 
all events. Those events are handled by the system so that a data storage can be constructed using those
events. Such a data storage could be a Oracle/MySQL/PostgreSQL database, some key-value store, a MongoDB 
instance or any combination you can think of that suits your needs. In the beginning you could even get away
with just managing the entire system in memory if you wanted to do that.

This is CQRS to it's essence: data altering commands should be separate from data queries so that you have
that freedom. Commands result in events and events alter the data of your system. That data can be built up as
 events change it, but it could be built on demand as well. Event handling could be transactional if needed in order
 to ensure consistency.

Most say CQRS systems are hard to implement due to the asynchronous nature of events and commands. But in most cases,
real-time data updates are rarely a requirement. So what if there is a 2 second delay between data change and the 
propagation of that change in all of the data storage systems? And even in those cases where real-time updates are needed,
you can build the system so that that particular part of the system behaves synchronously. If you take it a step further, you
could persist all your events as a data entity on it's own. This opens up huge possibilities for data analysis, as you can 
look into the individual operations that changed your data. Adding data auditing becomes a non-event. You can optimize for 
specific events. Once you start thinking in event rather than collections or tables, the possibilities become endless.

So it shouldn't be about whether your system can handle huge amounts of data.  It shouldn't be about whether your system is completely 
consistent all of the time. It should be about whether your system behaves reactively towards the user. Who cares if your system
is able to handle 500Gb of data if it's slow in handling data changes? Your system performance should be measured by it's capacity 
to reactively handle events, not the amount of data it can handle. Asynchronous data handling is, to me, the way to go. Your users really
won't kill you if you return a message stating that their data update request has been received and will be handled in a couple of seconds.
They will however rant when they have to stare at an 'UPDATING...' message for that same amount of time.
 
So stop storing data and start storing events. Build your data from those events, continuously or on demand. And whether you choose to use 
a NoSQL storage engine or an ACID-compliant SQL database is then entirely based on the various use cases your data consumers can come up with.
