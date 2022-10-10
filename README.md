# Connecting your NodeMCU with Adafruit to change the color of your LED over a wifi connection

## Introduction
In this manual you will learn how to connect your NodeMCU board with a wifi connection and your adafruit account to change the colour of your LEDstrip without having to rewrite the code.


## Required hardware
1. Arduino NodeMCU board
2. LEDstrip
3. 3 Jumpwires to connect to the LEDstrip


### Step 1: Connect your board with the LEDstrip

On the LED strip you see 3 jumpwire connections. These connections will have something written on them.

- 5v
- Din
- GND

Connect these 3 with the board 

- 5v  -> 5v
- Din -> D connections (Use D5 for this tutorial)
- GND -> GND


### Step 2 Installing the required Library
Go to **Sketch -> Include Library -> Manage Libraries** and search for 'Adafruit IO arduino' by 'Adafruit'. Select 'Install' and 'Install all packages'


![image](https://user-images.githubusercontent.com/74150653/194818761-3c2af94b-8a7c-4189-a880-ef44b3216ed8.png)


### Step 3: Adafruit

Go to Adafruit.com and make an account.
Go to Dashboard and make a new Dashboard named 'Color Picker'
Go to this dashboard and click on the settings icon


![image](https://user-images.githubusercontent.com/74150653/194819179-29257e27-d50b-436e-bc9b-581092f6422b.png)


Select '+ Create new block'


![image](https://user-images.githubusercontent.com/74150653/194819245-5be597eb-3b13-4405-bd43-2a31c43dbd84.png)


and select the color picker


![image](https://user-images.githubusercontent.com/74150653/194819294-17d93434-b249-40d2-afa5-1b8d08ba693c.png)


Name this block 'color' (**Important**!!)

To prepare for step 4, click on the yellow key icon and copy the Arduino codes.


![image](https://user-images.githubusercontent.com/74150653/194819589-22bd241b-76ad-458a-8793-ae566c081420.png)


## Step 4: The code
Go to **File -> Examples -> Arduino IO Adafruit -> Adafruit_14_neopixel** It will open a code with 2 tabs

```C++
// Adafruit IO RGB LED Output Example
//
// Adafruit invests time and resources providing this open source code.
// Please support Adafruit and open source hardware by purchasing
// products from Adafruit!
//
// Written by Todd Treece for Adafruit Industries
// Copyright (c) 2016-2017 Adafruit Industries
// Licensed under the MIT license.
//
// All text above must be included in any redistribution.

/************************** Configuration ***********************************/

// edit the config.h tab and enter your Adafruit IO credentials
// and any additional configuration needed for WiFi, cellular,
// or ethernet clients.
#include "config.h"

/************************ Example Starts Here *******************************/

#include "Adafruit_NeoPixel.h"

#define PIXEL_PIN     D5
#define PIXEL_COUNT   13
#define PIXEL_TYPE    NEO_GRB + NEO_KHZ800

Adafruit_NeoPixel pixels = Adafruit_NeoPixel(PIXEL_COUNT, PIXEL_PIN, PIXEL_TYPE);

// set up the 'color' feed
AdafruitIO_Feed *color = io.feed("color");

void setup() {

  // start the serial connection
  Serial.begin(115200);

  // wait for serial monitor to open
  while(! Serial);

  // connect to io.adafruit.com
  Serial.print("Connecting to Adafruit IO");
  io.connect();

  // set up a message handler for the 'color' feed.
  // the handleMessage function (defined below)
  // will be called whenever a message is
  // received from adafruit io.
  color->onMessage(handleMessage);

  // wait for a connection
  while(io.status() < AIO_CONNECTED) {
    Serial.print(".");
    delay(500);
  }

  // we are connected
  Serial.println();
  Serial.println(io.statusText());
  color->get();

  // neopixel init
  pixels.begin();
  pixels.show();

}

void loop() {

  // io.run(); is required for all sketches.
  // it should always be present at the top of your loop
  // function. it keeps the client connected to
  // io.adafruit.com, and processes any incoming data.
  io.run();

}

// this function is called whenever a 'color' message
// is received from Adafruit IO. it was attached to
// the color feed in the setup() function above.
void handleMessage(AdafruitIO_Data *data) {

  // print RGB values and hex value
  Serial.println("Received HEX: ");
  Serial.println(data->value());

  long color = data->toNeoPixel();

  for(int i=0; i<PIXEL_COUNT; ++i) {
    pixels.setPixelColor(i, color);
  }

  pixels.show();

}


```

Change the pin to **D5** and the count to the amount of led lights on your strip.

```C++
/************************ Adafruit IO Config *******************************/

// visit io.adafruit.com if you need to create an account,
// or if you need your Adafruit IO key.
#define IO_USERNAME  "THE_USERNAME_YOU_USED"
#define IO_KEY       "THE_KEY_GIVEN"

/******************************* WIFI **************************************/

// the AdafruitIO_WiFi client will work with the following boards:
//   - HUZZAH ESP8266 Breakout -> https://www.adafruit.com/products/2471
//   - Feather HUZZAH ESP8266 -> https://www.adafruit.com/products/2821
//   - Feather HUZZAH ESP32 -> https://www.adafruit.com/product/3405
//   - Feather M0 WiFi -> https://www.adafruit.com/products/3010
//   - Feather WICED -> https://www.adafruit.com/products/3056
//   - Adafruit PyPortal -> https://www.adafruit.com/product/4116
//   - Adafruit Metro M4 Express AirLift Lite ->
//   https://www.adafruit.com/product/4000
//   - Adafruit AirLift Breakout -> https://www.adafruit.com/product/4201
//   - Adafruit AirLift Shield -> https://www.adafruit.com/product/4285
//   - Adafruit AirLift FeatherWing -> https://www.adafruit.com/product/4264

#define WIFI_SSID "WIFI_NAME"
#define WIFI_PASS "PASSWORD"

// uncomment the following line if you are using airlift
// #define USE_AIRLIFT

// uncomment the following line if you are using winc1500
// #define USE_WINC1500

// uncomment the following line if you are using mrk1010 or nano 33 iot
//#define ARDUINO_SAMD_MKR1010

// comment out the following lines if you are using fona or ethernet
#include "AdafruitIO_WiFi.h"

#if defined(USE_AIRLIFT) || defined(ADAFRUIT_METRO_M4_AIRLIFT_LITE) ||         \
    defined(ADAFRUIT_PYPORTAL)
// Configure the pins used for the ESP32 connection
#if !defined(SPIWIFI_SS) // if the wifi definition isnt in the board variant
// Don't change the names of these #define's! they match the variant ones
#define SPIWIFI SPI
#define SPIWIFI_SS 10 // Chip select pin
#define NINA_ACK 9    // a.k.a BUSY or READY pin
#define NINA_RESETN 6 // Reset pin
#define NINA_GPIO0 -1 // Not connected
#endif
AdafruitIO_WiFi io(IO_USERNAME, IO_KEY, WIFI_SSID, WIFI_PASS, SPIWIFI_SS,
                   NINA_ACK, NINA_RESETN, NINA_GPIO0, &SPIWIFI);
#else
AdafruitIO_WiFi io(IO_USERNAME, IO_KEY, WIFI_SSID, WIFI_PASS);
#endif
/******************************* FONA **************************************/

// the AdafruitIO_FONA client will work with the following boards:
//   - Feather 32u4 FONA -> https://www.adafruit.com/product/3027

// uncomment the following two lines for 32u4 FONA,
// and comment out the AdafruitIO_WiFi client in the WIFI section
// #include "AdafruitIO_FONA.h"
// AdafruitIO_FONA io(IO_USERNAME, IO_KEY);

/**************************** ETHERNET ************************************/

// the AdafruitIO_Ethernet client will work with the following boards:
//   - Ethernet FeatherWing -> https://www.adafruit.com/products/3201

// uncomment the following two lines for ethernet,
// and comment out the AdafruitIO_WiFi client in the WIFI section
// #include "AdafruitIO_Ethernet.h"
// AdafruitIO_Ethernet io(IO_USERNAME, IO_KEY);
```
Paste the code you copied in **step 3** here:
#define IO_USERNAME 
#define IO_KEY 

and change:

#define WIFI_SSID "WIFI_NAME"
#define WIFI_PASS "PASSWORD"

To your wifi name and password (Make sure not to use a 5hz connection).


## Step 5: Lets play!
You did it! Now lets go to Adafruit and see if we can change the color.

Connect your board to your computer or a powersource, go to your adafruit dashboard and pick a color!


## Errors


![image](https://user-images.githubusercontent.com/74150653/194820738-2b780bba-3f88-4d03-b832-c2a241c2b6f0.png)



The only issue I walked into was the fact that I named 'color', 'kleur'. This resulted in a little frustration and not realizing this is why it didn't work. When I added a new block named 'color', this one did work and I deleted the previous one.

