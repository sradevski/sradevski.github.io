---
title: "Home Automation with Raspberry Pi"
layout: post
date: 2016-10-31 21:36
tag:
- home automation
- raspberry pi
- react native
- javascript
- nodejs
- python
image: /assets/images/jekyll-logo-light-solid.png
headerImage: true
projects: true
hidden: true # don't count this post in blog pagination
author: stevcheradevski
description: Automating your home is not as difficult as you might think. Using Raspberry Pi, some basic electronic circuit knowledge, and simple programming skills will lead you to an automated home.
jemoji: '<img class="emoji" title=":ramen:" alt=":ramen:" src="https://assets.github.com/images/icons/emoji/unicode/1f35c.png" height="20" width="20" align="absmiddle">'
externalLink: false
---

## Summary:

A detailed description of automating my small apartment to make my life a bit more convenient. The system is built using *Raspberry Pi*, some basic electronic circuit skills, *React Native* for a simple mobile app, and *Python* and *JavaScript* for the rest of the scripting involved. The end result is full control over the music player, air conditioner, and lights. They can be controlled either by a motion sensor, hardware buttons, or a mobile application.

## Introduction

It all started while talking with a friend about how cool it would be to turn on the aircon in winter from our labs, so when we get home it is warm and cozy. He already had a Raspberry Pi (RPI), so we just borrowed some LEDs, resistors and transistors from his lab and we made a simple prototype in 1-2 hours. After that I also got a Raspberry Pi, ordered some electronic parts, and we soldered a simple circuit that could receive and emit IR (infrared) signals. We successfully controlled the IR devices in our rooms, and the interface was very crude: ssh into the RPI and executing commands from the terminal. This is where my friend stopped, but I had fun working on it and continued by making it a bit more sophisticated.

 It is worth noting that I am far from an expert in electronics and everything I did was very new for me. All the references I used in making this system will be provided at the end of this post. I will assume that you have some basic knowledge of how RPI works and some basic programming skills.

## The Hardware:

Before discussing about the parts the system is compromised of, I will explain what I wanted to control with it. There were 3 IR-enabled devices, namely the air conditioner, an iPod dock, and the main light in the room. Moreover, I have 3 more lamps/spotlights that are connected to RF433 MHz-enabled sockets. Each of the devices came with their own remote controls. What I wanted is to be able to control all of these in ways I will explain later.

Let's start with the hardware. As I mentioned several times, the core of it is a Raspberry Pi (B+ in my case) device, running on Raspbian. Each of the electronic parts are depicted on the diagram below (except for a couple of resistors and a transistor), and I will explain how each of them is used.

As I didn't want to solder directly on the RPI, I used an adapter with the same pin distribution as the RPI, and soldered the circuit on it (see picture 1). This way the circuit can easily be removed and put back on, which is quite practical. I also recommend you to get a breadboard and cables as shown in the picture, so you can test and experiment without soldering anything. It is also a good idea to draw a diagram of the positioning of each electronic part before soldering. You also need to consider where your RPI will be located, in which direction the devices you want to control are (a must for IR-enabled devices), and place the electronic parts accordingly.

#### IR Receiver and Emitter

For all practical purposes, IR remotes emit an alternating sequence of on and offs with varying duration by turning the IR diode on and off. This IR signal is then received and decoded by an IR receiver, and then transmitted to a microcontroller as an electrical signal of ons and offs.

So, how do we know what signal do our remotes send? This is why we need an IR receiver. We simply point our remote at the receiver, press a button, and record the signal it emits. Then, when we want to execute the command for that button, we emit the specific sequence for that button through the IR emitter (diodes). Thanks to the LIRC tool for RPI, doing this is quite simple through the terminal.

There are basically two ways how remote controllers send signals. Each button has a specific code sequence, independent of the current state of the device; each button press sends the entire state of the remote. My music player and main light follow the first approach. The aircon follows the second approach (and the reason this is useful is that the displayed information on the remote control will always match the state of the aircon, assuming the last button press was received by the aircon). This is important because in the first approach case, you just record each button once, and then you make combinations in your scripts. With the second approach, however, you need to record an entire set of commands every time you want an additional option. So, to be able to set the temperature from 18 to 28 degrees in both hot and cold mode, you need to do 20 recordings as opposed to 11 recordings if the first approach is used.

In order to build the circuit and setup LIRC, just follow this (link required) detailed article, as there is no need to repeat the same things twice. Once you have it up and running, you are done for now.

#### RF433MHz Receiver and Emitter

In order to control my lamps/spotlights I used RF433MHz-enabled power sockets (see image 3). The 3 sockets were controlled by a remote control that came with them, so the goal was to replace it with my system. I thought in the beginning that I can follow the same approach as with the IR devices: record each button's signal from the remote, and then emit the recorded signal through an emitter.

Unfortunately, it didn't work. I tried recording the signal using 433Utils and pilight, and after several hours of frustration, I gave up on that approach. I am still not sure what the problem was, but it was impossible to distinguish between noise and the remote signal. The next approach was to reverse-engineer the remote control (so the receiver deemed unnecessary at the end).

What I did is I opened the remote control, and checked what kind of chip I had on my remote. I had some Chinese chip called HS2260A-R4. The next step is to find the datasheet for the chip you have on hand. The datasheet will have a detailed description of the protocol the chip uses to send data to the sockets. There are several things you need to learn about your chip, which hopefully everything will make sense by the end of this section.

The first thing you need to discover is the packet size that the remote uses to send commands to the sockets. In my case it was 12 tri-state "bits" plus a sync bit, but it seemed that the third, float state was not used. Of those, 8 were address bits (like an unique ID for each socket), and 4 were data bits. Then, you need to find out how big the pulse cycle is for one bit (mine was 2 cycles of 512 pulses for 1 bit).

Each bit is coded by some on/off durations, which you can find in the datasheet. In my case the "zero" was coded as "256 768 256 768" which basically says 256 oscillations on, 768 off, 256 on, 768 off. This will result in 1 tri-state bit. The "one" bit had a similar pattern, namely "768 256 768 256". At the end there is a sync bit, which was "256 8096" in my case. Note that the decoding in the sockets was not so sensitive, so even if you input "8000" it will still work.

Since my sockets were 12 bit + sync bit ones, the message looked like this:
pilight-send -p raw -c "768 256 768 256 768 256 768 256 256 768 256 768 256 768 256 768 256 768 256 768 256 768 256 768 256 768 256 768 256 768 256 768 768 256 768 256 768 256 768 256 768 256 768 256 768 256 768 256 256 8704"

In binary, this would be "110000001111" (sync bit not included).

In order to find the address and data for on and off for your sockets, it may take a bit of tinkering. Since my sockets had a “reset” button when pressed accepted any RF433 signal, I could set the addresses by myself. This made it much easier since I had to experiment with just 4 bits (16 combinations) to learn the data bits for on and off. I wrote a small script to generate all the combinations for the package, and run them one by one. I found that in my case, the data “0000” is to turn off, and “1111” is to turn on. In fact, not all bits are used, so more combinations will work the same (I was too lazy to find out which ones are used on mine).

If you can’t reset your sockets, just follow a brute-force approach. There are 4096 combinations in total (you can actually just try “0000” for the data bits, reducing the combinations to 256). Turn on the socket with your remote, and write a small script to try all addresses one by one with some delay, and see which one turns off the light.

In the beginning I used pilight library to test things out and then moved to 433Utils in "production" (and I don't remember what the reason was), but you can use either of them. Check the source code to see how I call the 433Utils tool (codesend C compiled file).

#### Motion and Temperature/Humidity Sensors

The motion and temperature/humidity sensors are quite simple. They are self-contained, so you simply connect them to a VCC (source), ground, and a GPIO pin, and they are good to go. The sensitivity and frequency of the motion sensor can be adjusted by turning the two potentiometers (check the specifications with your vendor). We will talk about the software controlling and using these two a bit later.

#### Hardware Buttons

The hardware buttons in my system were from a broken digital scale that I took apart. There is one clickable button and a 4-position switch. I have only 2 positions connected to GPIOs, and I make combinations of the switch + the clickable button that I will explain later in the software section.

## Raspberry Pi Software

The software of the RPI is compromised of scrpts controlling the input/output, and a nodeJS server used by the mobile application. I must say that the code is rather messy and the scripts are not that well organized, so it might not be as easy to reuse them. As everything works well for me, I haven't bothered to rewrite them (maybe in the future).

#### Scripts

The scripts are divided in 3 abstraction layers. Additionally, there is a config.json file, which is basically a persistent state storage.

The lowest abstraction layer is the core script. Its role is to abstract away the execution of IR-related commands, RF433MHz-related commands, and write and read the config file.

On top of the core, there a single script for each of the output devices to control (player, aircon, main light, and radio lights). Each script basically represents the API for that particular device, and they also handle the setting of the appropriate values for the config file.

The rest of the scripts are built on top of the "API". The input for the scripts is either from the input hardware (button, motion sensor), or from the terminal. Each script controls a single device or a combination of devices.

There are two types of scripts that are to be called from the terminal. The first one is controller scripts that control a single logical device, which are basically a wrapper over the API for each device (aircon controller, lights controller, player controller). The second type is more abstract, logical combinations of commands. The come home script dictates what should happen when I arrive home. The off all script takes care of turning everything off. The go to sleep script dictates what should happen before I go to sleep. These are either called from the terminal, or from the hardware input scripts, as I will explain now.

The PIR sensor script takes input from the motion sensor, and it executes the come home script when I arrive home. It does nothing while I am home. In order for the system to know if I am home or not, there is a field in the config denoting it (and I will explain later how it is set).

Even though the button switch has 4 positions, only 2 are connected to GPIOs. This means that there are 3 possible states for the switch (00, 01, 10). Depending on the switch state, and how long the clickable button is held down, there is a different command being executed. As there are a lot of combinations that happen, it is easier to just check the source code instead of me explaining in text.

There are two scripts that are not used in my current system. One is the *temperature sensor* script, which reads the current humidity and temperature and prints it out in the terminal. The temperature readings are not that accurate because of the proximity to the RPI and the precision of the sensor itself, and I sometimes am curious of the humidity in my room, but that is all the usage it gets. The second script that is not used is the *phone on network* script. I was hoping that I can periodically check all devices connected to the network, and if my phone is not on the list, it means I am not home (as I always have wifi on) and it can safely turn off everything. Unfortunately, when the phone is locked, it automatically turns off the wifi to preserve battery, and it didn't work as planned. The same thing could be achieved now with the nodeJS server polling the mobile app, but I haven't found the time to do it.

#### NodeJS Server



## React Native Mobile Application

## Conclusion

## References

Dependencies:

433Utils (the codesend compiled file comes from here): https://github.com/ninjablocks/433Utils
RPi.GPIO (used for getting input from the buttons): https://pypi.python.org/pypi/RPi.GPIO
LIRC (used for IR-related operations): http://www.lirc.org/

Useful tools:
wiringPi - nice console interface to test GPIOs.

Links regarding IR transmitter/receiver:
http://alexba.in/blog/2013/01/06/setting-up-lirc-on-the-raspberrypi/
http://www.instructables.com/id/Reverse-engineering-of-an-Air-Conditioning-control/?ALLSTEPS
http://www.ocinside.de/html/modding/linux_ir_irrecord_guide.html

Links regarding RF433 transmitter/receiver:
http://www.wes.id.au/2013/07/decoding-and-sending-433mhz-rf-codes-with-arduino-and-rc-switch/
https://wiki.pilight.org/doku.php/psend
https://github.com/sui77/rc-switch
http://stevenhickson.blogspot.jp/2015/02/control-anything-electrical-with.html

Links regarding the motion sensor:
https://www.raspberrypi.org/learning/parent-detector/worksheet/
https://www.modmypi.com/blog/raspberry-pi-gpio-sensing-motion-detection
https://www.mpja.com/download/31227sc.pdf

Links regarding the humidity/temperature sensor:
https://www.youtube.com/watch?v=IHTnU1T8ETk

Links regarding RPI settings:
https://www.raspberrypi.org/documentation/remote-access/vnc/README.md
https://www.raspberrypi.org/documentation/remote-access/ssh/unix.md
https://www.modmypi.com/blog/tutorial-how-to-give-your-raspberry-pi-a-static-ip-address

Links regarding installation of NodeJS on RPI:
http://unix.stackexchange.com/questions/207591/how-to-install-latest-nodejs-on-debian-jessie
