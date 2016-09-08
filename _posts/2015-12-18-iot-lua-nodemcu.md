---
layout: post
category : article
title: "IoT with Lua and the NodeMCU platform"
comments: true
tags : [iot]
---

Every couple of months it seems, there’s a new IoT platform emerging. Arduino, Raspberry Pi, Beaglebone Black… most of us have at least heard about them. Some time ago I wrote about the Particle Photon, which was another low-cost IoT device. Here we are and another platform has gotten more publicity: the NodeMCU.

One might think that due to the name, the NodeMCU would be an IoT device using Node.js but it isn’t. It’s a development development kit based on the ESP8266 module and allows you to code your device using Lua scripts. For those familiar with World of Warcraft coding, Lua will sound very familiar as it’s the scripting language used to write add-ons for the game.

However, Lua is also very suited for this use case. The terseness and clear syntax of the language makes it a very nice choice to code IoT applications. For example, to read or write from a GPIO pin, this code will do the trick:

{% highlight lua %}
pin = 1
gpio.mode(pin,gpio.OUTPUT)
gpio.write(pin,gpio.HIGH)
gpio.mode(pin,gpio.INPUT)
print(gpio.read(pin))
{% endhighlight %}

So what’s in the NodeMCU? It has built-in WiFi, an USB ports and has 10 GPIO ports. All pins can be configured as PWM, I2C, or other configurations, but not as analog pins. When you want to read analog input, you’ll need and ADC chip in order to read out values from analog sensors. Thanks to the WiFi, it makes connecting your IoT devices to the internet a whole lot easier.

The interesting thing is the price. If you know where to look, you can find NodeMCU development kit under $10. This makes it a very cheap prototyping option.

While not as feature complete as other platforms out there, it’s worth investigating. It’s small and given the fact that it has WiFi built in and support for a somewhat more readable language than an Arduino, it may suit your needs quite well. One of the downsides of the platform is the lack of documentation. The NodeMCU site does offer some examples, but you’ll mainly be relying on your own Google skill in order to find out how some parts happen. For a tinkerer, this is not an issue, but I wouldn’t recommend starting off using this platform. Given the fact that you need to piece together the information, you’ll probably end up taking just as much time as you would with an Arduino (but at a fraction of the cost).

My advice? With a price tag of $10, it’d order one just of the fun of it. This is one of those devices you’ll find a use case for anyway.
