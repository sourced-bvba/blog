---
layout: post
category : article
title: "Exploring the Particle Photon"
comments: true
tags : [iot]
---

So you want to get started with Arduino-style IoT but you’re not to keen on buying and assembling your Arduino and its shields? Well, here’s something for you that might spark your interest: the Particle Photon. The Photon is an fantastic attempt at making an entry level IoT platform, providing all the features an Arduino has with built-in WiFi functionaliteit and OTA updates. That’s right, you no longer need to connect your Arduino to your system to flash your code on the device (or use memory card tricks), but it can be done over the internet with virtually no setup.

But what is actually included in a Particle Photon? You get 18 digital IO interfaces (of which 9 support PWM), 8 analog ADC interfaces, 2 analog DAC interface, 2 SPI interfaces, an I2C interface, … Basically everything you need in an IoT platform. It support 802.11n WiFi and has a whopping 1Mb of flash memory available, so you can make huge sketches on these devices. It does draw on average 100 mA but has a minimum operating voltage of 3.6V, so a 3.7V LiPo will actually help you along quite nicely to power these devices. And the best part is its pricetag: a single Photon will set you back a whopping… $19! If you really want to go fancy, there’s a cellular version for $39 as well. Quite some bang for the buck, if you ask me.

In addition, due to the fact that every device is cloud enabled, you can use a REST API to query and modify the values of each of the IO interfaces. Every device is identified uniquely over the internet and can be accessed using a configurable access token. This makes interfacing with the device extremely easy and effortless.

But how hard is it to code a Photon? The language is based on Wiring, so it looks a lot like an Arduino sketch. However, some things have been added to support the REST functionality of the platform. Code is flashed to your device using a cloud based IDE.

For example, blinking a LED on D0 (digital IO port 0) looks like this:

``` c
int led1 = D0;  
void setup() {        
  pinMode(led1, OUTPUT);      
}             
void loop() {        
  digitalWrite(led1, HIGH);        
  delay(1000);        
  digitalWrite(led1, LOW);        
  delay(1000);      
}
```

If you connect a LED to D0 (using a small resistor), that’s it. You just blinked a LED using code you flashed from the internet.

Now, the real strength lies in enabling the REST features the board has to offer. If we want to for example toggle a LED through the REST interface, you can do this with the following code:

``` c
int led1 = D0;
void setup() {
  pinMode(led1, OUTPUT);
  Spark.function("led",ledToggle);
  digitalWrite(led1, LOW);
}
void loop() {
  // no looping, we're doing remote calls here!
}
int ledToggle(String command) {
    if (command=="on") {
        digitalWrite(led1,HIGH);
        return 1;
    }
    else if (command=="off") {
        digitalWrite(led1,LOW);
        return 0;
    }
    else {
        return -1;
    }
}
```

Now with a simple REST call we can turn the led on and off.

``` bash
# EXAMPLE REQUEST IN TERMINAL      
# Core ID is 0123456789abcdef      
# Your access token is 123412341234      
curl -X POST https://api.particle.io/v1/devices/0123456789abcdef/led \        
-d access_token=123412341234 \        
    -d params=on
```

Using the Photon, you can make truly connected devices, controlled by a REST interface. The downside is that if the internet is down, so is the interaction with your device. You can however run the cloud server yourself, but the documentation on that is still under construction and requires some experimentation.
