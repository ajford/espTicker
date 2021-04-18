# Arduino espTicker Library v4.x.x

The **Arduino espTicker Library** allows you to create easily espTicker callbacks, which can call a function in a predetermined interval. You can change the number of repeats of the callbacks, if repeats is 0 the ticker runs in endless mode. Works like a "thread", where a secondary function will run when necessary. The library use no interupts of the hardware timers and works with the **micros() / millis()** function. You are not (really) limited in the number of espTickers.

## New in v4.0
- added get interval function
- added remaining function
- added support for functional callbacks, only for ARM and ESP devices, e.g.<br> "Examples/FunctionalARM/FunctionalARM.ino"

## New in v3.1
- added set interval function

## New in v3.0
- radical simplified API
- generally you have to declare all settings in the constructor
- deleted many set and get functions
- if you need former functionality please use the version 2.1

## New in v2.1
- You can change the interval time to microseconds.
```cpp
espTicker tickerObject(callbackFunction, 100, 0, MICROS_MICROS) // interval is now 100us
```
- smaller improvments

## New in v2.0
- You can determine the number of repeats, instead of modes.
- The internal resolution is now **micros()**, this works with intervals up to 70 minutes. For longer intervals you can change the resolution to **millis()**.
```cpp
espTicker tickerObject(callbackFunction, 1000, 0, MILLIS)
```
- unified data types and smaller improvments

## Installation

1. "Download":https://github.com/sstaub/espTicker/archive/master.zip the Master branch from GitHub.
2. Unzip and modify the folder name to "espTicker"
3. Move the modified folder on your Library folder (On your `Libraries` folder inside Sketchbooks or Arduino software).


## How to use

First, include the TimerObject to your project:

```cpp
#include "espTicker.h"
```

Now, you can create a new object in setup():

```cpp
espTicker tickerObject(callbackFunction, 1000); 
tickerObject.start(); //start the ticker.
```

In your loop(), add:

```cpp
tickerObject.update(); //it will check the espTicker and if necessary, it will run the callback function.
```


## IMPORTANT
If you use delay(), the espTicker will be ignored! You cannot use delay() command with the TimerObject. Instead of using delay, you can use the espTicker itself. For example, if you need that your loop run twice per second, just create a espTicker with 500 ms. It will have the same result that delay(500), but your code will be always state.

## Example

Complete example. Here we created five timers, you can run it and test the result in the Serial monitor and the on board LED.

```cpp
#include "espTicker.h"

void printMessage();
void printCounter();
void printCountdown();
void blink();
void printCountUS();

bool ledState;
int counterUS;

espTicker timer1(printMessage, 0, 1); // once, immediately 
espTicker timer2(printCounter, 1000, 0, MILLIS); // internal resolution is milli seconds
espTicker timer3(printCountdown, 1000, 5); // 5 times, every second
espTicker timer4(blink, 500); // changing led every 500ms
espTicker timer5(printCountUS, 100, 0, MICROS_MICROS); // the interval time is 100us and the internal resolution is micro seconds


void setup() {
  pinMode(LED_BUILTIN, OUTPUT);
  Serial.begin(9600);
  delay(2000);
  timer1.start();
  timer2.start();
  timer3.start();
  timer4.start();
  timer5.start();
  }

void loop() {
  timer1.update();
  timer2.update();
  timer3.update();
  timer4.update();
  timer5.update();
  if (timer4.counter() == 20) timer4.interval(200);
  if (timer4.counter() == 80) timer4.interval(1000);
  }

void printCounter() {
  Serial.print("Counter ");
  Serial.println(timer2.counter());
  }

void printCountdown() {
  Serial.print("Countdowm ");
  Serial.println(5 - timer3.counter());
  }

void printMessage() {
  Serial.println("Hello!");
  }

void blink() {
  digitalWrite(LED_BUILTIN, ledState);
  ledState = !ledState;
  }

void printCountUS() {
  counterUS++;  
  if (counterUS == 10000) {
    Serial.println("10000 * 100us");
    counterUS = 0;
    }
  }
```

## Documentation

### States

```cpp
enum status_t {
  STOPPED,
  RUNNING,
  PAUSED
  };
```

### Constructors

```cpp
espTickers::espTickers(fptr callback, uint32_t timer, uint16_t repeats, interval_t mode)
```

Creates a espTicker object

- **callback** for the function name you want to call
- **timer** set the interval time in ms or us depending from mode
- **repeats** set the number of repeats the callback should executed, 0 is endless (default)
- **mode** set the interval resolution to MILLIS, MICROS_MICROS or MICROS (default)

**Example**

```cpp
espTickers timer(blink, 1000); // calls function blink() every second, internal resolution is micros, running endless
espTickers timer(blink, 1000, 5); // calls function blink() every second, internal resolution is micros, only 5 repeats
espTickers timer(blink, 1000, 0, MILLIS); // calls function blink() every second, internal resolution is millis, running endless
espTickers timer(blink, 1000, 0, MICROS_MICROS); // calls function blink() every 1000 microsecond, internal resolution is micros, running endless
```

### Destructor

```cpp
espTickers::~espTickers()
```
Destructor for espTicker object

## Class Functions

### espTicker Start

```cpp
void espTickers::start()
```

Start the espTicker. Will count the interval from the moment that you start it. If it is paused, it will restart the espTicker.

**Example**

```cpp
timer.start();
```

### espTicker Resume

```cpp
void espTickers::resume()
```

Resume the espTicker. If not started, it will start it. If paused, it will resume it. For example, in a espTicker of 5 seconds, if it was paused in 3 seconds, the resume in continue in 3 seconds. Start will set passed time to 0 and restart until get 5 seconds.

**Example**

```cpp
timer.resume();
```

### espTicker Pause

```cpp
void espTickers::pause()
```

Pause the espTicker, so you can resume it.

**Example**

```cpp
timer.pause();
```

### espTicker Stop

```cpp
void espTickers::stop()
```

Stop the espTicker.

**Example**

```cpp
timer.stop();
```

### espTicker Update

```cpp
void espTickers::update()
```

Must to be called in the main while() loop, it will check the espTicker, and if necessary, will run the callback.

**Example**

```cpp
while(1) {
  timer.update();
1.   }
```

### espTicker set Interval Time

```cpp
void espTickers::interval(uint32_t timer)
```

Changes the interval time of the espTicker. Depending from the mode it can millis or micro seconds.

- **timer** set the interval time in ms or us depending from mode


**Example**

```cpp
timer.interval(500); // new interval time
```

### espTicker get Interval Time

```cpp
uint32_t espTickers::interval()
```

Get the interval time of the espTicker. Depending from the mode it can millis or micro seconds.

**Example**

```cpp
uint32_t intervalTime;
intervalTime = timer.interval(); // get the interval time
```

### espTicker State

```cpp
status_t espTickers::state()
```

Returns the state of the espTicker.

**Example**

```cpp
status_t status;
status = timer.state();
```

### espTicker Elapsed Time

```cpp
uint32_t espTickers::elapsed()
```

Returns the time passed since the last tick in ms or us depending on mode.

**Example**

```cpp
uint32_t elapse;
elapse = timer.elapsed();
```

### espTicker Remaining Time

```cpp
uint32_t espTickers::remaining()
```

Returns the remaining time to the next tick in ms or us depending on mode.

**Example**

```cpp
uint32_t remain;
remain = timer.remaining();
```

### espTicker Counter

```cpp
uint32_t espTickers::counter()
```

Get the number of executed callbacks.

**Example**

```cpp
uint32_t count;
count = timer.counter();
```
