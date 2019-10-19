# CSEliteBadge2019
Reverse engineering the hackable badge from CyberStart Elite 2019.
## DISCLAIMER
This repo is for informational purposes only, and is sourced entirely from my own testing. None of this information is official, nor is endorsed, encouraged or supervised by Cyber Discovery, CyberStart, or any of their affiliates. Before you do anything, make sure your badge's flash memory is backed up as writing to the chip will **destroy** any program currently on the badge. If you break something, injure yourself or others, this is because you are stupid and you should feel bad. Not my fault.

By using this information you agree to these terms.

Please note that this page is in active development. If something is missing, give me a shout on Discord @DaCukiMonsta#4242, or on Twitter @hastwil, and I'll see if I can help. :)

## Contents
+ [Disclaimer](#disclaimer)
+ [Specs](#specs)
+ [Pinout](#pinout)
+ [LCD](#lcd)
+ [Uploading sketches](#uploading-sketches)

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
|3|LCD Chip Select (CS)|Low|
|4|SD Card Chip Select (CS)|Low|
|5|SPI MOSI/MCU Data Out||
|6|SPI MISO/MCU Data In||
|7|SPI SCK/Clock||
|8|Unknown/Not Connected||
|9|Unknown/Not Connected||
|10|Unknown/Not Connected||
|11|LED 1|High|
|12|LED 2|High|
|13|LED 3|High|
|14|LED 4|High|
|15|LED 5|High|
|16|Button UP|Low|
|17|Button DOWN|Low|
|18|Button LEFT|Low|
|19|Button RIGHT|Low|
|20|Button A|Low|
|21|Button B|Low|
|22|Unknown/Not Connected||
|23|Unknown/Not Connected||
|24|LCD Data/Command Select (RS/A0)||
|25|Unknown/Not Connected||
|26|LCD Reset (RST)|Low|
|27|LCD Backlight RED|High|
|28|LCD Backlight GREEN|High|
|29|LCD Backlight BLUE|High|
|30|Unknown/Not Connected||
|31|Unknown/Not Connected||

## LCD
The LCD display on the badge is a [Midas MCCOG128064B12W-FPTLRGB](https://uk.farnell.com/midas/mccog128064b12w-fptlrgb/display-lcd-graphic-128x64-fstn/dp/2664760). It has an ST7565P controller chip inside, which uses SPI. A TU2221A RGB backlight strip runs accross the bottom of the LCD, this has 4 connections to the board and is connected separately from the main LCD ribbon cable.

While the backlight itself is capable of displaying any RGB colour, the backlight driving circuitry on the board (transistors Q1, Q2 and Q3 which invert the signals from the MCU making the backlight active high) limits this to only 8 colours including black. Each colour can either be on or off. This is a hardware limitation, however in theory it could be possible to use some sort of PWM to changed the percieved brightness of each colour.

I could not find any LCD library which would support this LCD out-of-the-box, however it can be made to work with [@olikraus](https://github.com/olikraus)'s [u8g2lib](https://github.com/olikraus/u8g2) with some modifications. To do this, find the `u8x8_d_st7565.c` file under `~/Arduino/libraries/U8g2/src/clib`, and replace the block starting `static const uint8_t u8x8_d_st7565_zolen_128x64_init_seq[] = {` with the following:

```Processing
static const uint8_t u8x8_d_st7565_zolen_128x64_init_seq[] = {
    
  U8X8_START_TRANSFER(),             	/* enable chip, delay is part of the transfer start */
  
  U8X8_C(0x0e2),            			/* soft reset */
  U8X8_C(0x0ae),		                /* display off */
  U8X8_C(0x040),		                /* set display start line to 0 */
  
  //U8X8_C(0x0a1),		                /* ADC set to reverse */
  U8X8_C(0x0c8),		                /* common output mode */
  // Flipmode
  // U8X8_C(0x0a0),		                /* ADC set to reverse */ // this has been commented out for use with MCCOG128064B12W
  // U8X8_C(0x0c0),		                /* common output mode */
  
  U8X8_C(0x0a6),		                /* display normal, bit val 0: LCD pixel off. */
  U8X8_C(0x0a2),		                /* LCD bias 1/9 */
  U8X8_C(0x02f),		                /* all power  control circuits on (regulator, booster and follower) */
  U8X8_CA(0x0f8, 0x000),		/* set booster ratio to 4x */
  U8X8_C(0x027),		                /* set V0 voltage resistor ratio to max  */
  U8X8_CA(0x081, 0x013),		/* set contrast, contrast value, EA default: 0x016 */ // this has been changed to 0x013 for use with MCCOG128064B12W
  
  U8X8_C(0x0ae),		                /* display off */
  U8X8_C(0x0a5),		                /* enter powersafe: all pixel on, issue 142 */
  
  U8X8_END_TRANSFER(),             	/* disable chip */
  U8X8_END()             			/* end of sequence */
};
```

Now you can use the `U8G2_ST7565_ZOLEN_128X64_F_4W_HW_SPI u8g2(U8G2_R0, /* cs=*/ 3, /* dc=*/ 24, /* reset=*/ 26);` constructor for graphics and text, or the `U8X8_ST7565_ZOLEN_128X64_4W_HW_SPI u8x8(/* cs=*/ 3, /* dc=*/ 24, /* reset=*/ 26);` constructor for just text. However, this is not yet working perfectly. The image seems to be too far to the left with noise on the right-most few columns of the screen. Please note you should also include `<SPI.h>` at the start of your sketch.

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