--- 
layout: post
category : article
title: "Johnny Five, a nice introduction to Arduino"                
tags : [hobby]
---

Learning Arduino is not simple, especially if you're not C-minded. See, Arduino is programmed in C and I'm not afraid to say my C is quite rusty. However, there have been some new developments lately that enable you to program an Arduino with JavaScript.<!--more-->

Well, it's not actually programming. There is something called Firmata, which enables other systems to communicate and read/write analog and digital pins on an Arduino running the Firmata sketch. One of those systems is called [Johnny Five](https://github.com/rwaldron/johnny-five) (for those wondering where the name comes from, go and see Short Circuit). It's an Node.js and thus JavaScript based framework that interacts with Firmata so that you can read/write the pins on your Arduino. 

It took me a while to see the potential, but if you think about it and join a Raspberry Pi running Node.js with an Arduino with Firmata, there's not a lot you can't do. 

For my project, it's definitely worth exploring. There's one thing that I already know of that will cause problems and that's the fact that Firmata doesn't support software serial communication. My current design needs that to read the pH and EC sensors. However, this is only a minor issue, as it's possible to make my own circuit that reads a pH and EC sensor through an analog port. This would require me to do the calibration and soldering myself, but again, not that big an issue.

For the rest most of the things I need to do I can do with Johnny Five (controlling my peristaltic pumps and writing to the LCD). All in all, if I get the sensors to work correctly, it'll even be a lot easier as I should be able to integrate the Johnny-Five stack into the MEAN stack I wanted to use for my front-end. Since most of my logic will be on the Pi, this would also allow me to do this project with an Uno as the sketch for Firmata runs on an Uno.

As it stands, I'm going to give Johnny Five a shot. 
