---
layout: post
category : opinion
title: "On IT and elections"
comments: true
tags : [java]
---

Yesterday we had another big election day in Belgium. We had to vote for 3 levels of government (regional, federal and European). When it comes to voting, we're still a bit old-fashioned. A lot of voting bureaus still
work with (huge sheets of) paper and a red pencil. However, electronic voting is also present in more and more cities throughout the country for more than 10 years now I think.

You would think that they would have streamlined the system by now. Clearly, they didn't.<!--more--> One bug in the system created huge delays in tallying the votes, in another case the USB sticks were malfunctioning. The most blatant problem (for me as an IT professional) was the case of floppy disks malfunctioning. Yes, you read it correctly. Floppy. Disks. You know, the physical representation of the save icon. In 2014 the Belgian government is still relying on portable magnetic disks. Sigh.

If you take a step back, making a basic voting system isn't that hard. And making a decent electronic voting system in 2014 shouldn't be that much of an issue. I live in a country where you can pave the street with people who have a higher education degree. We have some of the best (and coincidentally more expensive) IT consultancy markets out there. You would think that we would have tackled this issue. You would think that by now we would have Raspberry Pi-based
electronic voting systems directly connected to the internet, running an ultra-secure version of Linux and which have triple data redundancy (cloud, local disk, usb storage) built in. You would think we would have an online system allowing out-of-country citizen to vote without having to wait for the ballots to get sent over, filling them in and sending them back (and this all needs to happen in a timespan of 12 days). Clearly, we don't.

Which brings us to the question: FOR THE LOVE OF EVERYTHING SACRED, WHY? 

Why indeed. Well, because it's the government. Nothing stiffles progress as much as bureaucracy, and of that we have plenty over here in Belgium (I've given up trying to explain our government structure to non-Belgians). Some of the most expensive IT projects which fail more than often can be found withing some level of our government. I don't want to think about all the tax euros that have been sunk into some of the most megalomanic systems they've come up with. It's a safe bet to say that if an IT project is initiated by the government, it'll either get scrapped due to budget overruns or goes so much over budget that it becomes too big to fail. In any case, we never seem to end up with the thing that was planned for (not even close).

That why I'm not surprised when I hear floppy disk failures caused tally delays. Our next election is due in 2018 and I'm sure we'll still be voting with paper and pencil or shoving data on a obsolete data carrier. The solution to the issue is a radical shift in how government IT projects are handled: small scale, agile and up-to-date. The only way a project like this is going to succeed is when the government is able to step away from its current process of handling IT projects. They need to look at the current technological champions (and we have a lot of them over here) and have the courage to aim for the creation of one of the best voting systems out there. But I don't see that happening any time soon.

I just hope that when my son gets to vote, I will get to tell him stories about how we used to vote and that he not gets to see it in action or in the news. And I hope I don't have to explain what a floppy disk is. Only 18 more years to go... One can hope.

Update 27-05-2014:

The source of the bug was identified. As I read it, it was a very basic bug that should have been caught during testing. This is one of the things our lead tester would have found immediately (he has detected much more unlikely things than this) and chances are that this would be already covered in a unit test of the code if we would have written it. According to the government, the software had passed certification and had undergone extensive testing. Apparently, standards don't seem to be that high. I wonder whether this certification process is open to the public?

Perhaps it would be an idea to make an open-source voting software platform where we could verify the quality of the system ourselves, a system that touches the very core of our society: democracy. We deserve such a system. We should demand such a system.
