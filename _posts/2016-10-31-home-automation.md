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
headerImage: false
projects: true
hidden: false # don't count this post in blog pagination
author: stevcheradevski
description: Automating your home is not as difficult as you might think. Using Raspberry Pi, some basic electronic circuit knowledge, and simple programming skills will lead you to an automated home.
externalLink: false
---

## Summary:

A detailed description of automating my small apartment to make my life a bit more convenient. The system is built using *Raspberry Pi*, some basic electronic circuit skills, *React Native* for a simple mobile app, *nodeJS* for a server, and *Python* for the rest of the scripting involved. The end result is full control over the music player, air conditioner, and lights. They can be controlled either by a motion sensor, hardware buttons, or a mobile application. You can find the source code [here](https://github.com/sradevski/homeautomate).


## Introduction

It all started while talking with a friend about how cool it would be to turn on the aircon in winter from our labs, so when we get home it will be warm and cozy. He already had a Raspberry Pi (RPI), so we just borrowed some LEDs, resistors, and transistors from his lab and we made a simple prototype in 1-2 hours. After that, I also got a Raspberry Pi, ordered some electronic parts, and we soldered a simple circuit that could receive and emit IR (infrared) signals. We successfully controlled the IR devices in our rooms, with a very crude interface: ssh into the RPI and execute commands from the terminal. This is where my friend stopped, but I had fun working on it and continued by making it a bit more sophisticated.

 It is worth saying that I am far from an expert in electronics, and everything I did was very new for me, except for some basics that I learned in high school. All the references I used in making this system will be provided at the end of this post. I will assume that you have some basic knowledge of how RPI works and some basic programming skills.

 ![Prototype](/assets/images/homeAutomate/prototype_breadboard.jpg)

## The Hardware:

Before discussing the parts the system is compromised of, I will explain what I wanted to control with it. There are 3 IR-enabled devices, namely the air conditioner, an iPod dock, and the main light in the room. Moreover, I have 3 more lamps/spotlights that are connected to RF433 MHz-enabled sockets. Each of the devices came with their own remote control. What I wanted is to be able to control all of these to make certain actions easier to perform (coming home, turning everything off at once, etc).

Let's start with the hardware. As I mentioned several times, the core of it is a Raspberry Pi (B+ in my case) device, running on Raspbian. The whole system architecture is depicted in the left diagram below. Each of the electronic parts is depicted on the right diagram below (except for a couple of resistors and a transistor), and I will explain how each of them is used.

![System Overview](/assets/images/homeAutomate/System.png "System Overview")
![System Parts](/assets/images/homeAutomate/Architecture.png "System Parts")

As I didn't want to solder directly on the RPI, I used an adapter with the same pin distribution as the RPI and soldered the circuit on it like [this one](/storage/projectsData/homeAutomate/pinheader.jpg). This way the circuit can easily be removed and put back on, which is quite practical. I also recommend you to get a breadboard and cables as shown on the prototype picture above, so you can test and experiment without soldering anything. It is also a good idea to draw a diagram of the positioning of each electronic part before soldering. You also need to consider where your RPI will be located, in which direction the devices you want to control are (a must for IR-enabled devices), and place the electronic parts accordingly.

#### IR Receiver and Emitter

For all practical purposes, IR remotes emit an alternating sequence of on and off with varying duration by turning the IR diode on and off. This IR signal is then received and decoded by an IR receiver, and then transmitted to a microcontroller as an electrical signal of ones and zeros (high and low voltage, to be more accurate).

So, how do we know what signal our remotes send? This is why we need an IR receiver. We simply point our remote at the receiver, press a button, and record the signal it emits. Then, when we want to execute the command for that button, we emit the same sequence for that button through the IR emitter (diodes). Thanks to the [LIRC tool](http://www.lirc.org/) for RPI, doing this is quite simple through the terminal.

There are basically two ways how remote controllers send signals. Each button has a specific code sequence, independent of the current state of the device; each button press sends the entire state of the remote. My music player and main light follow the first approach. The aircon follows the second approach (and the reason this is useful is that the displayed information on the remote control will always match the state of the aircon, assuming the last button press was received by the aircon). This is important because, with the first approach, you just record each button once, and then you make combinations in your scripts. With the second approach, however, you need to record an entire set of commands every time you want an additional option. So, to be able to set the temperature from 18 to 28 degrees in both hot and cold mode, you need to do 20 recordings as opposed to 11 recordings if the first approach is used.

In order to build the circuit and setup LIRC, just follow [this](http://alexba.in/blog/2013/06/08/open-source-universal-remote-parts-and-pictures/) article with a nice circuit diagram and explanation on how to setup LIRC, as there is no need to repeat the same things twice. Next, we continue with the other parts.

#### RF433MHz Receiver and Emitter

In order to control my lamps/spotlights I used RF433MHz-enabled power sockets like [these](/storage/projectsData/homeAutomate/remote_socket.jpg). The 3 sockets were controlled by a remote control that came with them, so the goal was to replace it with my system. I thought in the beginning that I can follow the same approach as with the IR devices: record each button's signal from the remote, and then emit the recorded signal through an emitter.

Unfortunately, it didn't work. I tried recording the signal using [433Utils](https://github.com/ninjablocks/433Utils) and [pilight](https://www.pilight.org/), and after several hours of frustration, I gave up on that approach. I am still not sure what the problem was, but it was impossible to distinguish between noise and the remote's signal. The next approach was to reverse-engineer the remote control (so the receiver deemed unnecessary at the end).

What I did is I opened the remote control, and checked what kind of chip I had on my remote. As the circuitry is quite simple, it should be too difficult to locate the chip. I had some Chinese chip called HS2260A-R4. The next step was to find the datasheet for the chip that I had. The datasheet has a detailed description of the protocol the chip uses to send data to the sockets. There are several things I had to learn about your chip, each used as a parameter for the tool I used for emitting signals (I ended up using *433Utils* at the end).

The first thing to discover is the packet size that the remote uses to send commands to the sockets. In my case, it was 12 tri-state "bits" plus a sync bit, but it seemed that the third, float state, was not used. Of those, 8 were address bits (like a unique ID for each socket), and 4 were data bits. Then, I had to find out how big the pulse cycle is for one bit (mine was 2 cycles of 512 pulses for 1 bit). Each cycle has an on and an off period with varying duration.

Each bit is coded by some on/off durations, which you can find in the datasheet. In my case, the "zero" was coded as "256 768 256 768" which basically says 256 oscillations on, 768 off, 256 on, 768 off. This will result in 1 tri-state bit. The "one" bit had a similar pattern, namely "768 256 768 256". At the end, there is a sync bit, which was "256 8096" in my case. Note that the decoding by the sockets is not so sensitive, so even if you input "8000" it will still work.

Since my sockets were 12 bit + sync bit ones, the message looked like this:

`pilight-send -p raw -c "768 256 768 256 768 256 768 256 256 768 256 768 256 768 256 768 256 768 256 768 256 768 256 768 256 768 256 768 256 768 256 768 768 256 768 256 768 256 768 256 768 256 768 256 768 256 768 256 256 8704"`

In binary, this would be "110000001111" (sync bit not included). Once you collect all this information, you need to adjust the parameters in the tool you use. If I remember correctly, I just changed the parameters in the source code of 433Utils and compiled the tool. I am sure that depending on the tool, you can also specify them as parameters.

In order to find the address and data for on and off for your sockets, it may take a bit of tinkering. My sockets had a “reset” button, which when pressed, the socket accepted any RF433 signal, so I could set the addresses by myself. This made it much easier since I had to experiment with just 4 bits (16 combinations) to learn the data bits for on and off. I wrote a small script to generate all the combinations for the package, and run them one by one. I found that in my case, the data “0000” is to turn off, and “1111” is to turn on. In fact, not all bits are used, so more combinations will work the same (I was too lazy to find out which ones are used on mine).

If you can’t reset your sockets, just follow a brute-force approach. There are 4096 combinations in total (you can actually just try “0000” for the data bits, reducing the combinations to 256). Turn on the socket with your remote, and write a small script to try all addresses one by one with some delay, and see which one turns off the light.

In the beginning, I used *pilight* library to test things out and then moved to *433Utils* in "production" (and I don't remember what the reason was), but you can use either of them. Check the source code to see how I call the *433Utils* tool (*codesend* C compiled file).

#### Motion and Temperature/Humidity Sensors

The temperature/humidity sensor is quite simple. I have a DHT11 sensor, which is less precise than DHT22, but I just wanted to test it out anyway, so it didn't matter so much. The sensor is self-contained, so you simply connect it to a VCC (source), ground, and a GPIO pin, and it is good to go.

The motion sensor is as simple as the temperature/humidity sensor. I am not sure what the model of my sensor was, I just ordered one online. The sensitivity and frequency of the motion sensor can be adjusted by turning the two potentiometers (check the specifications with your vendor).

We will talk about the how both of them are used (or not used) a bit later. You can check the source code for more details on how I read the data from each of them and how it is used afterwards.

#### Hardware Buttons

The hardware buttons in my system were from a broken digital scale that I took apart. There is one clickable button and a 4-position switch. I have only 2 positions connected to GPIOs, and I make combinations of the switch + pressing and holding the clickable button for different amount of time.

This is what the circuit looks like, without the buttons (it is not very pretty, I know):

![Circuit](/assets/images/homeAutomate/circuit.png "Finished Circuit")

## Raspberry Pi Software

The software of the *RPI* is compromised of scripts controlling the input/output, and a *nodeJS* server used by the mobile application. I must say that the code is rather messy and the scripts are not that well organized, so it might not be as easy to reuse them. As everything works well for me, I haven't bothered to rewrite them (maybe in the future).

#### Scripts

The scripts are divided into 3 abstraction layers. Additionally, there is a [config.json](https://github.com/sradevski/homeAutomate/blob/master/scripts/config.json) file, which is basically a persistent state storage.

The lowest abstraction layer is the [core script](https://github.com/sradevski/homeAutomate/blob/master/scripts/remote_core.py). Its role is to abstract away the execution of IR-related commands, RF433MHz-related commands, and write and read the config file.

On top of the core, there a single script for each of the output devices to control ([player](https://github.com/sradevski/homeAutomate/blob/master/scripts/player.py), [aircon](https://github.com/sradevski/homeAutomate/blob/master/scripts/aircon.py), [main light](https://github.com/sradevski/homeAutomate/blob/master/scripts/main_light.py), and [radio lights](https://github.com/sradevski/homeAutomate/blob/master/scripts/radio_lights.py)). Each script basically represents the API for that particular device, and they also handle the setting of the appropriate values for the config file.

The rest of the scripts are built on top of the "API". The input for the scripts is either from the input hardware (button, motion sensor), mobile app, or from the terminal. Each script controls a single device or a combination of devices.

There are two types of scripts that are to be called from the terminal. The first one is controller scripts that control a single logical device, which is basically a wrapper over the API for each device ([aircon controller](https://github.com/sradevski/homeAutomate/blob/master/scripts/aircon_controller.py), [lights controller](https://github.com/sradevski/homeAutomate/blob/master/scripts/lights_controller.py), [player controller](https://github.com/sradevski/homeAutomate/blob/master/scripts/player_controller.py)). The second type is more abstract, logical combinations of commands. The [come home](https://github.com/sradevski/homeAutomate/blob/master/scripts/come_home.py) script dictates what should happen when I arrive home. The [off all](https://github.com/sradevski/homeAutomate/blob/master/scripts/off_all.py) script takes care of turning everything off. The [go to sleep](https://github.com/sradevski/homeAutomate/blob/master/scripts/go_to_sleep.py) script dictates what should happen before I go to sleep. These are either called from the terminal, the mobile app, or the hardware input.

The PIR sensor script takes input from the motion sensor, and it executes the *come home* script when I arrive home. It does nothing while I am home. In order for the system to know if I am home or not, there is a field in the config denoting it. This field is set to false (not home) whenever the *off all* script is executed, with few minutes of delay (so I have time to get out of the house). The *off all* script is either executed through the terminal, the hardware buttons, or the mobile app.

Even though the button switch has 4 positions, only 2 are connected to GPIOs. This means that there are 3 possible states for the switch (00, 01, 10). Depending on the switch state, and how long the clickable button is held down, there is a different command being executed. As there are a lot of combinations that happen, it is easier to just check the source code instead of me explaining in text.

There are two scripts that are not used in my current system. One is the [temperature sensor](https://github.com/sradevski/homeAutomate/blob/master/scripts/temp_sensor.py) script, which reads the current humidity and temperature and prints it out in the terminal. The temperature readings are not that accurate because of the proximity to the RPI and the precision of the sensor itself, and I sometimes am curious of the humidity in my room, but that is all the usage it gets. The second script that is not used is the [phone on network](https://github.com/sradevski/homeAutomate/blob/master/scripts/phone_on_network.py) script. I was hoping that I can periodically check all devices connected to the network, and if my phone is not on the list, it means I am not home (as I always have wifi on) and it can safely turn off everything. Unfortunately, when the phone is locked, it automatically turns off the wifi to preserve battery, and it didn't work as planned. After making a nodeJS server, the same thing can be achieved by it polling the mobile app, but I haven't found the time to do it.

#### NodeJS Server

The *nodeJS* server, also not particularly well written, is very simple and it serves only the mobile application. It doesn't do authentication, validation, or anything sophisticated really. There is no need for authentication currently, as the RPI can only be accessed through the local network. It just has 5 post URIs that can be called from the mobile app independently. The call simply wraps the parameters sent from the mobile app, runs the appropriate script, and returns a config JSON with the latest state of the system when the script has finished executing. There are few more things happening in the code, but that is the essence of it. Because of the many constraints on the university VPN there cannot be a server hosted for outside access, and that is the reason why it works the way it does.

## React Native Mobile Application

As you can see on the screenshot below, the mobile application is also quite simple. Since I developed it using *React Native*, it works on both *android* and *ios*. The app is compromised of 4 independent modules, plus 3 buttons for the most common actions I do every day. As I have shortly explained about the server, each button press makes a call to the server with the target state, the server executes the command, the latest state is returned, and it is showed on the screen, thus providing feedback if the action succeeded or not. The alarm is simply turning on the music player at the chosen time, and optionally the aircon 30 minutes before the alarm turns on (so it is all cozy and nice when I wake up). Each interaction with the app communicates with the server and it keeps everything up to date. Also as mentioned above, in order for the app to work, it must be connected to my local wifi network.

![Mobile App](/assets/images/homeAutomate/mobile_app.png "Mobile App")

## Conclusion

Despite all the hacks and messy code, poor electronics knowledge, and a short period of time, I have managed to automate most of the electric appliances in my room. As the world of IoT (the most overused term in the past year) is advancing, only your creativity will be the limit of what you can do with almost no money and some knowledge in electronics and programming. Things can get much more complicated, of course, but starting simple is the way to go. It is also a great way to learn new things, while making your life a bit easier (or at least you will feel cool and you can show off to your friends). I urge you to do even the simplest of things, since there is nothing more rewarding than knowing a bit more every day, and seeing that something that you have created in a working state.

![Final Result](/assets/images/homeAutomate/finished_system.png "The Final Results")

## References

**Dependencies:**

[433Utils](https://github.com/ninjablocks/433Utils) (the codesend compiled file comes from here). <br>
[RPi.GPIO](https://pypi.python.org/pypi/RPi.GPIO) (used for getting input from the buttons). <br>
[LIRC](http://www.lirc.org/) (used for IR-related operations). <br>
[wiringPi](http://wiringpi.com/) (nice terminal interface to test GPIOs). <br>
[Adafruit DHT](https://github.com/adafruit/Adafruit_Python_DHT) (used to read the temperature/humidity sensor) <br>

**Links regarding IR transmitter/receiver:**

<http://alexba.in/blog/2013/01/06/setting-up-lirc-on-the-raspberrypi/> <br>
<http://www.instructables.com/id/Reverse-engineering-of-an-Air-Conditioning-control/?ALLSTEPS> <br>
<http://www.ocinside.de/html/modding/linux_ir_irrecord_guide.html> <br>

**Links regarding RF433 transmitter/receiver:**

<http://www.wes.id.au/2013/07/decoding-and-sending-433mhz-rf-codes-with-arduino-and-rc-switch/> <br>
<https://wiki.pilight.org/doku.php/psend> <br>
<https://github.com/sui77/rc-switch> <br>
<http://stevenhickson.blogspot.jp/2015/02/control-anything-electrical-with.html> <br>

**Links regarding the motion sensor:**

<https://www.raspberrypi.org/learning/parent-detector/worksheet/> <br>
<https://www.modmypi.com/blog/raspberry-pi-gpio-sensing-motion-detection> <br>
<https://www.mpja.com/download/31227sc.pdf> <br>

**Links regarding the humidity/temperature sensor:**

<https://www.youtube.com/watch?v=IHTnU1T8ETk> <br>

**Links regarding RPI settings:**

<https://www.raspberrypi.org/documentation/remote-access/vnc/README.md> <br>
<https://www.raspberrypi.org/documentation/remote-access/ssh/unix.md> <br>
<https://www.modmypi.com/blog/tutorial-how-to-give-your-raspberry-pi-a-static-ip-address> <br>

**Links regarding installation of NodeJS on RPI:**

<http://unix.stackexchange.com/questions/207591/how-to-install-latest-nodejs-on-debian-jessie> <br>
