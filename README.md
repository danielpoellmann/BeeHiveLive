# BeeHiveLive
This respository describes a bee hive monitor that is integrated to Home Assistant as a zigbee end device using ESPHome. The bee hive monitor is a custom made PCBA that includes an I2C interface for temperature/humidity measurement, load cell for weight measurement and solar charging circuitry for Lithium-Ion batteries.
> [!Note]
> It is almost 2am at night and my brain is not properly working anymore, so I would like to apologize for this documentation.
> Also there might be some sarkasm inside this document since the whole process makes me question my sanity.
>
> Let's see where this will get us...

## Software Development
> I have no idea how to programm so I basically just steal code and torture every AI known to man to get my code running.
> It is the software equivalent of using Duck Tape in order to mount a jet engine to a Boeing 787-9 Dreamliner.

### Step 1: Being a failure at using the nrf52840
I planned to use a seeed studio brakeout board based microcontroller that features Zigbee connectivity.
That enables me to connect the device to my already established Zigbee mesh.
After a short search I stumbled apon twoe different options:
1. Seeed Studio XIAO ESP32C6: https://wiki.seeedstudio.com/xiao_esp32c6_getting_started/
2. Seeed Studio XIAO NRF52840: https://wiki.seeedstudio.com/XIAO_BLE/

After some googling "ESP32C6 vs NRF52840 current consumption" it turns out, that the NRF52840 is supposed to have a really low power comsumption, making it absolutely perfect for my application.
Even something about compatibility with ESP Home and the Arduino IDE is written in the documentation, so we know that we will have an easy time programming it (Foreshadowing can be quite obvious).

#### The limitations of ESP Home and yaml
It is possible to generate code in ESP home and it actually has such a low power consumption, that I was not even able to pick it up with my USB power meter.
However there is one problem. I want to be compatible with One-Wire devices, I2C and a HX711 weight sensor.
At first I tried implementing one of my dallas temperature sensors using the One-Wire protocoll.
I fired up the code generation in ESP Home and after exactly 93.82 seconds my professional "ESPHome-YAML-Developer" career came to an end:
<img width="2010" height="98" alt="image" src="https://github.com/user-attachments/assets/df24ccdc-0d2e-4281-a9fc-826220a76eed" />

I am not going to pretend to know what happened here, but my friend who works as a copilot for microsoft knows whats up.
There is currently no library for the usage of one wire devices with the NRF52840 chip in ESP Home.
At least it looks like smarter people than me are also aware of that problem: https://github.com/esphome/esphome/issues/16163

So let's just scrap the ESP Home approach for now and use something thats lets me use my full c-programming potential. After all, ESP Home might just be dragging me down (might include sarcasm).

#### The Savior? - An avaliable library for NRF52840 in the Arduino IDE
The last time I used arduino was back in universtity. There was only a white background for the IDE which caused me to delete it.
After all a real developer needs a dark mode and lines of text flying through a terminal.

However since the Arduino IDE is aimed towards beginners like me a wanted to give it a try.
Indeed there is a library avaliable to get us started with my specific NRF52840 board. 
So I added the board URL to my preferences and tried out example sketches.
<img width="994" height="663" alt="image" src="https://github.com/user-attachments/assets/d85e8646-102b-4b85-9245-276259d98ea5" />
> [!Important]
> I am not going to paste the URL here, since this approach will not work anyways and we are running straight against a wall here. Unforfunately my past self doesn't know this yet and will spend 4 hours of debugging and looking through forums before finally giving up.

It turns out that programming zigbee to the nrf52840 is not possible in the Arduino IDE. Again my copilot friend from microsoft told me this.
Appearently since some version of the nRF Connect SDK, zigbee is not supported anymore for the nrf52840 chip.
I did not even try to do the same thing in PlatformIO for Visual Studio Code, since there I was greeted with two issues:
1. For connectivity only bluetooth shows up
2. I cannot even find my Seeed Studio XIAO nrf52840 board
<img width="1349" height="102" alt="image" src="https://github.com/user-attachments/assets/1d6770c4-67f7-4ed2-9a86-60dcbefad9e8" />

#### Going full professional since the Arduino IDE is for beginners
I guess after 14hours without sleep infront of three monitors with one of them only showing music videos, I can call myself professional developer. I somehow feel the urge to go to a TED talk and tell people about my journey.

Back to topic: So the recommended way by the people who built the nrf52840 is the usage of Visual Studio Code + nRF Connect SDK.
Awesome stuff and let me tell you: Now the real pain begins.

In fact I had to use Ibuprophen 400 just to keep myself motivated.
> [!Warning]
> This is just a joke. Don't do drugs, it will ruin your code.

I played around with the environment and found out the following facts:
1. Zigbee is not avaliabe in every SDK version -> v2.9.2 seems to have the required libraries
2. It is extremely important where you create your project folder

When buliding code the automatically generated folder structure grows a lot. So far in fact that the code generation will fail due  to the amount of symbols in the filepath (now we know the maximum is 250 symbols).

3. Zigbee example projects do not allow the usage of the Seeed Studio board with its bootloader

The Seeed Studio version acts like a usb drive. When you double press reset, it shows up in the explorer and allows you to drag .uf2 files to it. These are not generated by the examples by default.
> [!Caution]
> This next step fried my bootloader and I do not have the required hardware to flash a new one

Fortunately (I thought to myself) it is possible to bulid them by adding this line in the prj.conf file:
```
CONFIG_BUILD_OUTPUT_UF2=y
```
I pulled the .uf2 into my nrf52840 wich caused the chip to freeze and in addition I cannot even flash a blinky sketch anymore.
My guess to what happened is, that I never defined how this uf2 file is supposed to be structured.
For comparison, this is how a .uf2 file is supposed to look like:
<img width="551" height="27" alt="image" src="https://github.com/user-attachments/assets/5c7e2f65-90aa-47ec-8651-97c2d2a9a1bd" />

And this is what my sleep deprived brain came up with:
<img width="530" height="26" alt="image" src="https://github.com/user-attachments/assets/d36a2f4b-3699-43ad-ba6a-de24c32204c3" />

I wonder what is usually stored between 0x00000 and 0x27000...

In the next few hours I tried to get the blinky sketch running on the nrf52840. It took me quite some time to realize my problem, since the board still shows up as a USB device when I double-click the reset button. It ignores every .uf2 file I try flashing on it. I even wondered at some point if my built-in LED is just broken and so I hooked up a logic analyzer to the D0 Pin to check if it will toggle:
<img width="2111" height="189" alt="image" src="https://github.com/user-attachments/assets/5737b3a4-8dfd-4967-be1f-efa5b4fd9ab9" />

Of course it doesn't...

Let's try to destroy the ESP32C6 next!

### Step 2: ESP32C6 is easy to programm if only I knew how to do it
