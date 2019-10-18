# CDEliteBadge2019
Reverse engineering the hackable badge from Cyber Discovery Elite 2019.
## DISCLAIMER
This repo is for informational purposes only, and is sourced entirely from my own testing. None of this information is official, nor is endorsed, encouraged or supervised by Cyber Discovery or any of their affiliates. Before you do anything, make sure your badge's flash memory is backed up as writing to the chip will **destroy** any program currently on the badge. If you break something, injure yourself or others, this is because you are stupid and you should feel bad. Not my fault.

By using this information you agree to these terms.

Please note that this page is in active development. If something is missing, give me a shout on Discord @DaCukiMonsta#4242, or on Twitter @hastwil, and I'll see if I can help. :)
## Specs
+ Microcontroller: [ATMega1284P](http://ww1.microchip.com/downloads/en/devicedoc/doc8059.pdf)
+ LCD: [Midas MCCOG128064B12W-FPTLRGB](https://uk.farnell.com/midas/mccog128064b12w-fptlrgb/display-lcd-graphic-128x64-fstn/dp/2664760)

## Pinout
Here is the pin to device mappings for the microcontroller chip on the board.

| MightyCore/Arduino Pin  | Function           | Active High/Low |
|-------------------------|--------------------|-----------------|
|0|Unknown/Not Connected||
|1|Unknown/Not Connected||
|2|Unknown/Not Connected||
|3|LCD Chip Select (CS)|H|
|4|Unknown/Not Connected||
|5|SPI MOSI/MCU Data Out||
|6|SPI MISO/MCU Data In||
|7|SPI SCK/Clock||
|8|Unknown/Not Connected||
|9|Unknown/Not Connected||
|10|Unknown/Not Connected||
|11|LED 1|H|
|12|LED 2|H|
|13|LED 3|H|
|14|LED 4|H|
|15|LED 5|H|
|16|Button UP|L|
|17|Button DOWN|L|
|18|Button LEFT|L|
|19|Button RIGHT|L|
|20|Button A|L|
|21|Button B|L|
|22|Unknown/Not Connected||
|23|Unknown/Not Connected||
|24|LCD Data/Command Select (RS/A0)||
|25|Unknown/Not Connected||
|26|LCD Reset (RST)|L|
|27|LCD Backlight RED|H|
|28|LCD Backlight GREEN|H|
|29|LCD Backlight BLUE|H|
|30|Unknown/Not Connected||
|31|Unknown/Not Connected||

## Uploading sketches
Currently, there is no known way to upload sketches to the board without using the unpopulated ICSP header `J5` on the rear of the badge.

**Uploading via USB is broken at this stage, and writing to the chip over USB (even through avrdude) will likely break the on-board programmer. This seems to be a glitch/mistake with the FTDI chip, because this is supposed to work.**

To program over ISCP, you may want to solder a header onto the badge. If you know how to solder and understand the risks (of pulling traces up, etc) then go ahead. Otherwise, maybe wait for a USB method to be developed.

You will need an Arduino with a USB interface (using an UNO as an example here) and you should upload the ArduinoISP sketch (found in Arduino IDE -> File -> Examples -> 11. ArduinoISP) to the board. Make sure the power switch is OFF (so the batteries aren't connected to external power) and the badge isn't plugged in over its own USB port.  Then, connect the badge to the Arduino as below.

|Badge `J5` Header Pin|Arduino Pin (UNO Example)|
|-|-|
|VCC|3.3V|
|GND|GND|
|MISO|MISO (12)|
|SCK|SCK (13)|
|RESET|10|
|MOSI|MOSI (11)|

Next, install [MightyCore](https://github.com/MCUdude/MightyCore#boards-manager-installation), and set the following values under the Tools menu in the Arduino IDE:
+ Board: `ATMega1284`
+ Variant: `1284P`
+ Programmer: `Arduino as ISP`
+ Port: `(whichever serial port your Arduino (not the badge) uses)`

To upload, press Ctrl-Shift-U or the equivalent for your system. Pressing Ctrl-U or Upload in the IDE will upload to the Arduino, not the badge. We want to use the Arduino as a programmer, not as the board to be programmed, so pressing shift lets the IDE know this.
