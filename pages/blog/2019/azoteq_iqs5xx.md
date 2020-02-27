title: Azoteq IQS5xx ESP8266 Compatibility
date: 2019-07-29
published: true 
summary: Minor edits to some provided example code
tags: embedded, arduino, i2c

#Working with the Azoteq ProxSense Trackpads

I've been prototyping out a few home automation projects over the last few months, and I've found a few uses for the [Azoteq ProxSense](https://www.azoteq.com/images/stories/pdf/proxsense_i2c_trackpad_datasheet.pdf) trackpad modules. They're small capacitive touchpads with the ability to track coordinates and certain types of gestures, and interface easily over I2C. I ended up trying to talk to one with an ESP8266 running Arduino firmware. If you're familiar with the ESP8266, you'll know that rather than a hardware I2C implementation, it uses a software one. This is just fine, generally, and gets abstracted away by the Arduino libraries, except if someone like Azoteq releases their [sample code](https://www.azoteq.com/design/software-and-tools/) for Arduino using the AVR TWI hardware registers -___-. So, replacing functions that refer to the hardware registers is necessary if you're lazy like me and would like to leverage their reference implementation, in this case with functions from the higher level Wire library. Anyone still reading this has probably come across this exact fact, and so here are some replacement functions for you: <https://gist.github.com/mrT-F/e2b6a0f4f4e4d9c53a2af53a89070193>

There's only three functions that actually have to get replaced, the rest is mostly retry logic and other wrappers, so you can just replace those functions and the sample project should work fine. You'll have to define your IQS5xx_SDA and IQS5xx_SCL pins as well.

The code quality is average at best, and has absolutely not been rigorously reviewed, but you may do absolutely anything you'd like with it.