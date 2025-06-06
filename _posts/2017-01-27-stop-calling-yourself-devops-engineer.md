---
layout: post
category : article
title: "Stop calling yourself a DevOps engineer"
comments: true
tags : [rant]
---

So this is an article about something that's been bothering me for a very
long time. It's about the guys that call themselves "DevOps engineers". There's no
such thing. There, I said it.

Let me explain. For those that actually know what DevOps is, they know that it's
not a role. It's a mentality. It's in the same range as Agile. I work in Agile
environments all the time, but you don't see me saying I'm an AgileDev. No, I'm a
Java dev that happens to love Agile.

The current batch of "DevOps engineers" are infrastructure engineers. Plain and simple. It's not because
you suddenly automated your infrastructure writing Puppet or Chef recipes that you're
suddenly a developer. It's not because you suddenly abandoned physical servers for virtual ones in the cloud and
are able to the AWS web console that you're suddenly doing something revolutionary that requires a new title.
Unless you're actually writing production code that's directly affecting the business user in any other way than a infrastructure engineer would or
let's just call it what it is, which is Ops, you're Ops. Period. You're not "a DevOps engineer".
There's no such thing!

DevOps is a mentality. It's development
working closely together with Ops as an integrated unit in all aspects of the complete lifecycle of software. And in that regard,
if DevOps truly is at the heart of your company, you won't probably even have a lot of Ops people anymore.
In that case, your developers will probably use DevOps techniques in order to simply
replace most of the tasks a "DevOps engineer does". Puppet isn't rocket science only able to be
understood by those who know their way around hardware and operating systems, nor is Chef.

Or to quote Gene Kim, the guy who wrote The Phoenix Project and The DevOps Cookbook:

> Currently, DevOps is more like a philosophical movement, and not yet a precise collection of practices, descriptive or prescriptive (e.g., CMM-I, ITIL, Agile, etc.).  At this early stage we’re in, DevOps is more like a vibrant community of practitioners who are interesting in replicating the performance outcomes and culture as exemplified in the seminal John Allspaw/Tim Hammond 2009 Velocity presentation about doing “ten deploys a day” at Flickr.

The ideal situation in an organisation embracing DevOps is where the code of the developer
also contains the code for the infrastructure. Infrastructure as code, sound familiar?
This is also where I'm leaning to: developers not only deliver the code that delivers the business value,
they also provide the needed scripts to provision the servers that will run my code, automate the
monitoring tools and alerts. Why? Because that's how they'll deploy their code locally as well.
Releases and production deploys become the most benign thing in the world and not something to be
afraid of, because it's been done so many times before. A production deploy is just a push of
a button. It's a bleak future for new "DevOps engineers", because basically, Dev just stole the bulk of their fun
work and they're back to... just plain boring Ops. Ouch.

As for me, I don't understand why suddenly every hipster Ops suddenly called himself (or herself)
a DevOps engineer the moment they started writing scripts to automatically create their servers
on Amazon AWS. No, well in some way, I do. By calling themselves DevOps and protecting their newly minted role,
code in production with the touch of a button, adding servers on the fly. I've yet to see a "DevOps engineer" explaining to Devs
how their provision scripts work, or pair with a Dev to write a Chef recipe or any other script that runs on a server for that matter.
I can't speak for all Devs, but I love to pair with an Ops to get an idea of the impact we have on Ops (like monitoring, logging, ...). What I have seen multiple times is "DevOps engineers" still grouped together in their own corner, away from the Dev department, surrounded by hardware,
having the same tools on their TV screens like they did when they were just Ops, sending specs to developers on how they should log something in the log file.

You're an Ops. I'm a Dev. At most we both work in a DevOps company if we work together in the same team as an integrated unit. DevOps is a set of Dev and IT Ops practices that work in unison in order to create a better environment for everyone to work in and to provide better quality to the clients.

But none of us are "DevOps engineers". Stop calling yourself that.
