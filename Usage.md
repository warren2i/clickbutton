#Installing and usage of the ClickButton library



# Installing #

For Arduino 0017 onwards, just extract the contents of the archive into the "libraries" folder located in your sketchbook folder.


---


# Syntax #
To instantiate a ClickButton object named _buttonObject_
```
ClickButton buttonObject(pin [,active [,CLICKBTN_PULLUP]]);
```

where:
  * buttonObject is your name for the button object in code.
  * pin is the pin connected to the button
  * active denotes an active LOW or HIGH button (default is LOW)
  * CLICKBTN\_PULLUP turns on the internal pullup resistor. This is only possible with active low buttons.



---


# Notes of notable warnings #

<font color='FF0000'>
<h2>Avoid using delay()!</h2>
</font>

It stops everything (except interrupts) that the Arduino does. Then the button timings may get delayed(!) too much, and the library will not work as intended.

### Also avoid using a "home-made" delay-without-delay kind of delay !! ###
It doesn't really matter if you use the built-in `delay()` function, or if you make, say a _for_-loop, _while_-loop or similar, just wasting time and CPU cycles.. it's still a form of delay. In which case I'd say it's likely that you should rethink your "game engine" (or main() loop).

Instead, have a look at [Blink without delay](http://arduino.cc/en/Tutorial/BlinkWithoutDelay) and [AvoidDelay](http://playground.arduino.cc/Code/AvoidDelay) on the Arduino site for more.


## Avoid using it from inside interrupts ##
By "it" I mean this library. ISR / Interrupts stops most other things in the micro-controller (Arduino), including the timer counter obtainable from the _millis()_ function. ClickButton is rather dependent on this timer..



---




# Functions #
(Note the _buttonObject_ is just a placeholder for any object name you use, of course)

## Update() ##
The only function.

Type: void (returns nothing).

```
buttonObject.Update();
```

Reads the button, debounces it, and updates it's click count.
This should basically be read once each main program loop.

Note that the click count is lost after the button is released and the Update() function is called.


---


# Public variables / members #


## .clicks ##
(formerly ".click" - without the "s")

Returns the button click count (also possible to set as well, but that wouldn't seem too useful).

This will be reset to zero after the button is released and another call to the Update() is made, so you probably need to save this value for later (See example code at the bottom of page).

```
buttonClicks = buttonObject.clicks
```


### Returned click counts: ###

  * A <font color='0000FF'><b>positive</b></font> number returns the number of short clicks.
  * A <font color='FF0000'><b>negative</b></font> number indicates long clicks (or possibly click-and-hold).

"Short" clicks is simply the number of button clicks within a set timelimit (the _.multiclickTime_ public variable. Default 250 ms, or 0.25 seconds).

"Long clicks" are really just the last click that is held down for a "longer" time (Default 1 second). The preceding clicks, if present, are simply just short clicks. The returned absolute value is the number of clicks (including the last button hold-down).

A "click-and-hold" can be determined simply by getting a "long click" and then checking that the button is still held down (via the _.depressed_ public variable. See the "LEDfader" example that comes with the library).



## .depressed ##
Returns the currently debounced button (press) state.

  * true = button is pressed down
  * false = button is not pressed down.

This value is independent of button logic (active high or low).

Intended use is for a "click-and-hold" function, by checking if a button is still held down after a long click. But it also serves as an immediate, debounced button state.

```
if (buttonClicks == -1 && buttonObject.depressed == true) then its-still-held-down;
```



## .debounceTime ##
Sets / gets the time limit for ignoring button bounces.

If multiple clicks / signals are received within this time limit, they are ignored (So, do not set this too high or too similar to multiclickTime. Also, do not double-click faster than this :P)

```
buttonObject.debounceTime = 20;   // debounce time set to 20 milliseconds
```



## .multiclickTime ##
<font color='#777777'>(Yes, it's spelled like that, no capitalization on 'click'.. since I was thinking of 'multiclick' as one word. Might be a bit counterproductive, but for the time being that's what it is)</font>

Sets / gets the time limit for clicks.

If you want to do multiple clicks (like a double click), you must do it within this time limit. Default is set to 250 ms (0.25 seconds).

This also goes for single clicks, so this will be the delay determining when the single (or multiple) clicks are registered.

```
buttonObject.multiclickTime = 250;
```



## .longClickTime ##
(Formerly ".heldDownTime")

Sets / gets the minimum time for a "long click" (holding down the button for an extended time).

The returned click count will be negative (indicating a long click), and the absolute nr. is the number of (short) clicks right before and including the last long button press.

Default long click minimum held-down time is set to 1 second.

```
buttonObject.longClickTime = 1000;  // Sets minimum long click held-down time to 1 second
```


---



# MultiClicks example sketch #
This example is included with the library. Using an active-low button, and using the internal pull-up resistor of the Atmega chip.

```
/* ClickButton library demo

  Blinks a LED according to different clicks on one button.
  
  Short clicks:

    Single click - Toggle LED on/off
    Double click - Blink      (Toggles LED 2 times/second)
    Triple click - Fast blink (Toggles LED 5 times/second)
    
  Long clicks (hold button for one second or longer on last click):
    
    Single-click - Slow blink   (Toggles LED every second)
    Double-click - Sloow blink  (Toggles LED every other second)
    Triple-click - Slooow blink (Toggles LED every three seconds)


  The circuit:
  - LED attached from pin 10 to resistor (say 220-ish ohms), other side of resistor to GND (ground)
  - pushbutton attached from pin 4 to GND
  No pullup resistor needed, using the Arduino's (Atmega's) internal pullup resistor in this example.

  Based on the Arduino Debounce example.

  2010, 2013 raron
 
 GNU GPLv3 license
*/

#include "ClickButton.h"

// the LED
const int ledPin = 10;
int ledState = 0;

// the Button
const int buttonPin1 = 4;
ClickButton button1(buttonPin1, LOW, CLICKBTN_PULLUP);

// Arbitrary LED function 
int LEDfunction = 0;


void setup()
{
  pinMode(ledPin,OUTPUT);  

  // Setup button timers (all in milliseconds / ms)
  // (These are default if not set, but changeable for convenience)
  button1.debounceTime   = 20;   // Debounce timer in ms
  button1.multiclickTime = 250;  // Time limit for multi clicks
  button1.longClickTime  = 1000; // time until "held-down clicks" register
}


void loop()
{
  // Update button state
  button1.Update();

  // Save click codes in LEDfunction, as click codes are reset at next Update()
  if (button1.clicks != 0) LEDfunction = button1.clicks;
  

  // Simply toggle LED on single clicks
  // (Cant use LEDfunction like the others here,
  //  as it would toggle on and off all the time)
  if(button1.clicks == 1) ledState = !ledState;

  // blink faster if double clicked
  if(LEDfunction == 2) ledState = (millis()/500)%2;

  // blink even faster if triple clicked
  if(LEDfunction == 3) ledState = (millis()/200)%2;

  // slow blink (must hold down button. 1 second long blinks)
  if(LEDfunction == -1) ledState = (millis()/1000)%2;

  // slower blink (must hold down button. 2 second loong blinks)
  if(LEDfunction == -2) ledState = (millis()/2000)%2;

  // even slower blink (must hold down button. 3 second looong blinks)
  if(LEDfunction == -3) ledState = (millis()/3000)%2;


  // update the LED
  digitalWrite(ledPin,ledState);
}
```