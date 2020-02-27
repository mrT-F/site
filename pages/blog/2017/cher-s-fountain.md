title: Cher's Fountain
date: 2017-05-28
published: true
summary: An arduino-based water fountain for my cat
tags: arduino, electronics, cat, hardware

#Cats Hate Standing Water

I'm going to try to make an effort to document some of my hobby projects - mostly small electronics. This is a good way for me to remember how I designed something when it breaks later... I recently started building a water fountain for my cat. As evidenced by my [home page](/), I spend far too much time catering to her every whim. Anyway, I thought it would be fun to have her drink running water, rather than a standard water bowl. For simple automation tasks like this, I can't recommend SparkFun's [Arduino Pro Mini 328](https://www.sparkfun.com/products/11113) enough. Obviously, for a small project like this, you don't want to invest in a $30 arduino board. The Pro Mini board is less than $10, and can be programmed just like an arduino. There's a little soldering involved, but that's more fun anyway.

###Materials
* Soldering Iron
* [Arduino Pro Mini 328](https://www.sparkfun.com/products/11113)
* [Peristaltic Pump](https://www.adafruit.com/product/1150)
* [LED Button](https://www.sparkfun.com/products/10442)
* [1N4001 Diode](https://www.adafruit.com/product/755)
* [BC337 Transistor](https://www.sparkfun.com/products/13689)
* Some Resistors

###Getting Started

To start with, you'll need to get comfortable with programming the microcontroller board. I won't go into details - SparkFun has a great guide [here](https://learn.sparkfun.com/tutorials/using-the-arduino-pro-mini-33v). I suggest buying a few of these at a time - they're cheap and easy to drop into all kinds of projects. I soldered male headers onto one. I use that one for quick prototyping on a breadboard. Then, once I've verified everything's working, I'll solder wires directly into a second Pro Mini board. Here's a new, untouched one along with my prototyping board. 

![boards](/static/blog/2017/megaBoards.jpg)

Programming these is straightforward using Sparkfun's [FTDI Breakout Board](https://www.sparkfun.com/products/9873).

###Design

Including the button would give us an easy way to turn the water on or off. I also may or may not be trying to teach my cat to use the button. This particular button is not very well documented on SparkFun - I had to dig a little to figure out which pin was which. It turns out that of the four pins, two are the anode (A) and cathode (C) for the LED, and two are either side of a switch, B1 and B2, which is connected when the button is pressed. SparkFun has a [breakout board](https://www.sparkfun.com/products/10467) which can can help you identify which pin is which. I added a pull-up resistor to B2. In my case, this was a 6.8KΩ resistor, but anything near that should be fine. This would keep B2 high until the button was pressed, at which point it would go low. Add a little bit of logic to the arduino to read that change, and we have our on/off button.

The arduino can then put out a high signal when it knows the button is on. This is where we need to use the transistor. The pump I have is a 12V DC motor, so the transistor helps us output a 12V signal. If you're using different materials, read the datasheet for the motor and make sure the transistor you use is rated correctly for the motor. This transistor is rated at 50V and 800mA, while this motor is 12V and 300mA, so we're good here. Ensure you have a current-limiting resistor between the arduino output and transistor base - this could prevent damage to the transistor. Arduino output pins should be limited to 40mA anyway I believe, but better safe than sorry.

I also added a diode across the motor pins. This prevents voltage spikes when we turn the motor off. 

![diode pin](/static/blog/2017/motorDiode.jpg)

Here's the full circuit diagram. The only thing that's not included here is the LED light on the push button.

![circuit](/static/blog/2017/Circuit.png)

The ease of programming the board is the best thing about these Pro Mini boards. The following program was all the logic required to read the button and control the button LED and motor.


    //global pins
    const int buttonPressPin = 8;
    const int buttonStatePin = 7;
    const int motorPin = 3;
    int buttonRead = 0;
    int lastButtonRead = 0;
    int buttonState = 0;


    void setup() {
      // pin 8 is input from button - goes low when pressed
      //buttonStatePin is a software switch controlled by button press
      pinMode(buttonPressPin, INPUT);
      pinMode(buttonStatePin, OUTPUT);
      pinMode(motorPin, OUTPUT);
    }

    void loop() {
      // Sample on/off button - no real debouncing but works pretty well
      int buttonRead = digitalRead(buttonPressPin);
  
      if (buttonRead != lastButtonRead){
        //if we went from high to low - button press; invert LED
        if( buttonRead == LOW) {
          buttonState = ~buttonState;
          digitalWrite(buttonStatePin, buttonState ); 
          //start motor if turned on
          if (buttonState) {
            analogWrite(motorPin, 255);
          }
          //stop motor if turned off
          else {
            analogWrite(motorPin, 0);
          }
        }
        //delay a little to avoid bouncing
        delay(50);
      } 
      lastButtonRead = buttonRead;
 
    }

The analogWrite() call could be used to vary the voltage driving the motor and get variable pump rate. The pump already isn't especially strong though, so I don't think it's really worth doing.  

After I prototyped the design on a breadboard, and verified everything was working as expected (never a sure thing), I soldered the components and wires together on the "production" Pro Mini board. Before casing:

![final](/static/blog/2017/beforeCasing.jpg)

Nothing too impressive - I'll update with a picture once I find appropriate casing and bowl to give my cat some running water.

