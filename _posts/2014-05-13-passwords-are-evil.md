---
layout: post
category : opinion
title: "Passwords are evil"
comments: true
tags : [security]
---

Recently one of my good friends [@Ayame__](https://twitter.com/Ayame__) pointed me to a [article](https://medium.com/building-things-on-the-internet/a0c3eb525200) on Medium. In short, it's about what developers should stop doing when creating application for the internet.What I wanted to discuss is the second item on the list: asking people to create complex passwords. <!--more-->

I do believe that the author has a point: enforcing special characters on passwords isn't user friendly. And thinking your passwords is more secure when you're using special characters is a false sense of security. His example is simple, but goes to the core of the problem: a 7-character passwords using the most commonly used special characters is less secure than a 8-character password using only uppercase and lowercase letters. 

But what the author is missing completely is the fact that by omitting special characters, people will be inclined to use a combination of words and therefore opening up the possibility of a dictionary attack (which is a lot easier to do than brute force). But as the author states, even a 20-character passwords is hacked nowadays through social engineering. Btw, if the hackers are able to steal the entire database of a website like the author also states, the developers should be brought out back and shot for not storing password hashes.

My issue nowadays is not with passwords, it's the mere fact that passwords shouldn't be used anymore. There are numerous more secure alternatives to authentication than passwords that should be promoted and, dare I say it, even enforced over the usage of something as insecure as a password. Two-factor security is dead simple nowadays, with OTP applications like Google Authenticator or Duo Mobile. Hardware solutions like a YubiKey are readily available and dead simple to use. Especially in corporate environments: I'm still baffled by how much passwords are used to grant access to the most secure systems. Most of the times, in such environments, you won't have to look far for drawers containing post-its with the current password or some type of password-storage tool on a PC that contains every password, only to be secured by a single password (which coincidentally is easy to remember).

But the user is not to blame. We, as developers, have enforced passwords on them for the last couple of decades. We steadily ignore the fact that most of our users have a smartphone which can be used for authentication. If two-factor security was being used more often, things like HeartBleed would be page 11 news instead of a screaming front-page header. We're relunctant to adopt better security measures (no, social logins isn't more secure). We're afraid what our users might say if we try to enforce a better system to secure their data.

My plight is to decrease our dependency on passwords. We should still allow passwords, but we sure as hell shouldn't make it easy on the user to use passwords. Instead, we should make it easy to use safer authentication mechanisms. A user can still choose to use passwords, but then he'll have to live with the fact that it'll have to be a 15-character password that's not breakable by a basic dictionary attack. However, if he chooses to use an OTP tool, he could be allowed a more basic password. For internet applications, this is something for the long haul (but should be aimed for nonetheless). However, for enterprise and corporate applications, this should be enforced as quickly as possible. For example, here in Belgium, we all have an electronic ID card. However, its usage for authentication is mainly limited to applications by the government (despite the fact that implementing eID security is really, really easy). Things like eID, YubiKey, smartphone authentication or other OTP systems should be considered before even thinking about using basic passwords when implementing corporate security.

People want their data secure, but at the same time don't want to be bothered by the security measures put in place in order to do so. It a simple case of not being able to have the cake and eat it too. Things like HeartBleed make our dependency on passwords painfully obvious: I can't even remember the amount of 'please reset your password' mails in the aftermath. The worst thing of it all is that most of us simply comply. If someone suddenly found a security flaw in tumbler locks throughout the world there would be a lot of angry people. But in the digital world, which now holds a lot more valuable information that most things in your house, we're a lot more naive and complacent. And I'm afraid it'll take something more serious than HeartBleed to wake us up.