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

"DevOps engineers" are infrastructure engineers. Plain and simple. It's not because
you suddenly automated your infrastructure writing Puppet or Chef recipes that you're
suddenly a developer. Unless you're actually writing production code that's directly
affecting the business user in any other way than a infrastructure engineer or
let's just call it what it is, which is Ops, you're Ops. Period. You're not "a DevOps engineer".
There's no such thing!

DevOps is a mentality. It's development
working closely together with Ops as an integrated unit in all aspects of the complete lifecycle of software. And in that regard,
if DevOps truly is at the heart of your company, you won't probably even have a lot of Ops people anymore.
In that case, your developers will probably use DevOps techniques in order to simply
replace most of the tasks a "DevOps engineer does". Puppet isn't rocket science only able to be
understood by those who know their way around hardware and operating systems, nor is Chef.

The ideal situation in an organisation embracing DevOps is where the code of the developer
also contains the code for the infrastructure. Infrastructure as code, sound familiar?
This is also where I'm leaning to: developers not only deliver the code that delivers the business value,
they also provide the needed scripts to provision the servers that will run my code, automate the
monitoring tools and alerts. Why? Because that's how they'll deploy their code locally as well.
Releases and production deploys become the most benign thing in the world and not something to be
afraid of, because it's been done so many times before. A production deploy is just a push of
a button. It's a bleak future for new "DevOps engineers", because basically, Dev just stole the bulk of their fun
work and they're back to... just plain boring Ops. Ouch.

To me, I don't understand why suddenly every hipster Ops suddenly called himself (or herself)
a DevOps engineer the moment they started writing scripts to automatically create their servers
on Amazon AWS. Well, in some way, I do. By calling themselves DevOps and protecting their newly minted role,
they're making sure we don't go anywhere near Puppet or Chef and get the power to just deploy our
code in production with the touch of a button, adding servers on the fly. I've yet to see a "DevOps engineer" explaining to Devs
how their provision scripts work, or pair with a Dev to write a Chef recipe or any other script that runs on a server for that matter.
I can't speak for all Devs, but I love to pair with an Ops to get an idea of the impact we have on Ops (like monitoring, logging, ...). What I have seen multiple times is "DevOps engineers" still grouped together in their own corner, away from the Dev department, surrounded by hardware,
having the same tools on their TV screens like they did when they were just Ops, sending specs to developers on how they should log something in the log file.

You're an Ops. I'm a Dev. At most we both work in a DevOps company if we work together in the same team as an integrated unit.

But none of us are "DevOps engineers". Stop calling yourself that.
