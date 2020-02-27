title: DragonRise Inc. Generic USB Joystick Configuration with RetroPie
date: 2017-06-26
published: true 
summary: How to manually add a controller configuration to your emulation setup 
tages: raspberry pi, retropie, n64, usb joystick

#Retropie Controller Configuration

I recently set up a raspberry pi to run several classic game emulators - mostly for n64 games. Luckily, the great people contributing to the [RetroPie](https://github.com/retropie/retropie-setup/wiki/First-Installation) project make this basically a one-command, painless process. The emulators installed on my basic Raspbian build without any problems whatsoever. However, I ran into some issues when attempting to connect generic, third party n64 controllers to my new system. I bought a couple of these [n64 USB controllers](https://amazon.com/Classic-Retro-N64-Bit-Wired-Controller/dp/B00KBAE52S/) hoping they'd work out of the box.

Of course, they didn't. The emulators recognized the controllers as "DragonRise Inc. Generic USB Joystick" and RetroPie's built-in controller configuration refused to cooperate at all. The D-Pad and analog stick register as movements across the same axis, and RetroPie does not like that at all. This doesn't really matter for the n64 gameplay (generally), but RetroPie was not happy. A quick google search showed me that many people had the same problem, and there didn't seem to be much of a solution on any of the forums I looked at.

So, on to manual tools. I installed the "joystick" package with:

```
sudo apt-get install joystick
```

And then ran ```jstest /dev/input/js0``` command with one controller plugged in. You should see something like this:

![jstest](/static/blog/2017/jstest.png)

Now play with the buttons -- look! It lets you see what the RetroPie is actually registering for each button press or joystick toggle. Take note of these. From reading the RetroPie documentation, it seemed that the controller configuration was contained in the ```/opt/retropie/configs/``` folder. There is an ```InputAutoCfg.ini``` file within each emulator folder. This file contains default controller configuration that gets dynamically copied to the emulator configuration on emulator startup. By manually creating editing the configuration here, I was able to get my controllers to work seamlessly with all the n64 games I've tested so far. Here is my configuration section for the controllers in ```/opt/retropie/configs/n64/InputAutoCfg.ini```:

    ; DragonRise Inc.   Generic   USB  Joystick  _START
    [DragonRise Inc.   Generic   USB  Joystick  ]
    plugged = True
    plugin = 2
    mouse = False
    AnalogDeadzone = 4096,4096
    AnalogPeak = 32768,32768
    DPad R = axis(0+)
    DPad L = axis(0-)
    DPAD D = axis(1+)
    DPAD U = axis(1-)
    Start = button(9)
    C Button U = button(0)
    C Button R = button(1)
    C Button D = button(2)
    C Button L = button(3)
    B Button = button(4)
    A Button = button(5)
    L Trig = button(6)
    R Trig = button(7)
    Z Trig = button(8)
    X Axis = axis(0-, 0+)
    Y Axis = axis(1-, 1+)
    ; DragonRise Inc.   Generic   USB  Joystick  _END

Even if your controller is slightly different, you should be able to create a similar file by using ```jstest```. I'm posting this here mainly because I had a tough time finding any good documentation on this process, and it took me *way* longer than it should have to figure out this relatively simple configuration change. Also - obligatory statement that downloading ROMs you don't own the rights to is bad, mmkay?


