+++
title = "Ergodox Infinity LCD Firmware"
author = ["Walker Griggs"]
date = 2017-03-21
categories = ["devlogs"]
draft = false
creator = "Emacs 27.2 (Org mode 9.4.4 + ox-hugo)"
weight = 2009
+++

So you've got yourself an Ergodox Infinity. Congratulations! Everyone probably thinks your a little bit crazy spending that much on a keyboard that strange with LCD displays that small and a layout you're struggling to type on. But it's ok -- anyone who shares this strange obsession probably understands.

This post is really to demonstrate how to change the default layer's LCD logo. [Asciipr0n](http://asciipr0n.net/ergodox-infinity-logo/) has a very clean guide to this, but I find that parts of it are (if not the majority of it is) out of date. Since the firmware has been updated, I thought I'd update the guide.


## Prerequisites {#prerequisites}

I don't want to go too deep into these. Essentially, here is the shopping list of the things you'll need...


### Firmware {#firmware}

The firmware, and really the whole reason for this post, well be using is the [kiibohd/controller](https://github.com/kiibohd/controller). Jacob Alexander (aka Haata) is not only Input Clubs head honcho, but he IS Input Club (well... sorta). He not only wrote kiibohd, but also wrote kll (the keyboard layout language). You'll want to clone his firmware...


### dfu-util {#dfu-util}

This toolchain is what we'll be using to flash our firmware onto the board. I downloaded mine from apt-get but it's also available on Homebrew. It's simple enough to download.


### gcc-arm-none-eabi {#gcc-arm-none-eabi}

This one may only apply to me, but I feel like it shouldn't go unsaid. I needed to download the gcc-arm-none-eabi package to properly build the arm firmware with the gcc compiler. Granted, I'm running Debian over here, so you OSX users may not need this step.


### Python Imaging Library {#python-imaging-library}

This is only necessary if you plan to use kiibohd's bitmap2Struct.py conversion file. Custom logos can only be flashed in the form of byte array, so this script it highly recommended... unless you want to write your byte array by hand. Download 'Image' with pip...


## Customize Layout {#customize-layout}

So now that we have everything we need to continue, customize your layout. I just use [Input Club's Configurator](https://configurator.input.club/). It's quite simple and doesn't require too much explanation. Just select the button you want to change, and choose its new function. Go as deep into the layering as you wish. My one recommendation: keep a FLASH button on each half in layer seven. This way, you wont have to flip over your board and hit the reset button with paperclip.

Once you have everything mapped out, download the firmware from the configurator and set aside the ZIP file for later.

If you have aversion to this configurator, so be it. You can use whatever program --or lack thereof if you hate yourself -- you want, as long as the .kll files compile in the end


## Create a Logo {#create-a-logo}

This part is fun and quite straight forward. Create a logo that fits inside 128x32 screen. Anything large won't get flashed. You can create a the logo in any way, as long as you can get it to .bmp file. Originally, I used [Piskel](http://www.piskelapp.com/) to create mine.

{{< figure src="/img/ergodox-infinity-lcd-firmware/game_of_life.png" width="50%" >}}

I created the permutation of a glider from Conway's Game of Life. If you don't know exactly what that is, I highly recommend looking into it.

Essentially, the bitmap can be whatever so long as it's a black foreground on white background. (Though... I've just begun to tinker with and observe the conversion of color bitmaps to the monochromatic lcd display... So you can always give that a try).

Now in order to flash this new logo onto your board, it needs to be in the form of a byte array. The easiest way to convert your bitmap into the byte array is to use the firmware's [bitmap2Struct.py](https://github.com/kiibohd/controller/blob/master/Scan/Devices/STLcd/bitmap2Struct.py) -- as I mentioned earlier. This script spits out two visual representations of the bitmap and the byte array. Just shove the output into a file for later.

```bash
python bitmap2Struct.py --filename <filename> > ByteArray.txt
```

Here is what my ByteArray.txt file look like:

```nil
uint8_t array[] = {
0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0xf0, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x0f, 0x00, 0x00, 0x00, 0x00,
}
```


## Prepare the Firmware {#prepare-the-firmware}

Now that we have all of our files ready to go, it's time to prep the firmware. A few things have changed in the structure of the firmware, so it does take a few steps to get setup. Oddly enough, we need to build the default ergodox firmware in order to rebuild ours later.

```bash
cd controller/Keyboards
./ergodox.bash
```

Now you may notice in the firmware's root directory, a 'kll' directory has been created. That is where we need to add our custom layouts. So make yourself a layout directory and copy in all our .kll files from the ZIP the configurator created.

```bash
mkdir controller/kll/layouts/<my_layout>
cp <configurator ZIP>/*.kll controller/kll/layouts/<my_layout>
```

Since we have our logo's byte array all squared away, all we have to do is include it. Head into the Scan directory and copy the infinity_ergodox module.

```bash
cd controller/Scan
cp -r Infinity_Ergodox Infinity_Ergodox_Custom
```

Now the one and only thing we need to alter in here is the STLcdDefaultImage in scancode_map.kll. Replace the default Input Club's byte array with our custom byte array from earlier.

Bingo. Now our layouts are almost ready to be flashed. We now need to quickly modify our own build script.

```bash
cd controller/Keyboards && cp ergodox.bash ergodox-custom.bash
```

Edit this new bash file and update the DefaultMap and PartialMaps to include each layer's .kll map created in the configurator. You can also alter the BuildPath, but I'm not building more than one set of firmware at a time, so I leave them as the default ICED-L and ICED-R. Do note: each map (default or partial) requires the lcdFuncMap. Here is mine for example:

```bash
# This is the default layer of the keyboard
# NOTE: To combine kll files into a single layout, separate them by spaces
# e.g.  DefaultMap="mylayout mylayoutmod"
DefaultMap="<my_layout>/MDErgo1-Default-0 lcdFuncMap"

# This is where you set the additional layers
# NOTE: Indexing starts at 1
# NOTE: Each new layer is another array entry
PartialMaps[1]="<my_layout>/MDErgo1-Default-1 lcdFuncMap"
PartialMaps[2]="<my_layout>/MDErgo1-Default-2 lcdFuncMap"
PartialMaps[3]="lcdFuncMap"
PartialMaps[4]="lcdFuncMap"
PartialMaps[5]="lcdFuncMap"
PartialMaps[6]="lcdFuncMap"
PartialMaps[7]="<my_layout>/MDErgo1-Default-7 lcdFuncMap"
```

Finally, change the ScanModule from Infinity_Ergodox to Infinity_Ergodox_Custom or whatever you called your Scan Module. Now we should be all ready to flash.


## Build and Flash {#build-and-flash}

Now that we have everything set and ready to go, we can actually get this firmware onto your board and have you on your way. First step, rebuild the default firmware from earlier, but run your custom build script this time.

```bash
cd controller/Keyboards
./ergodox-custom.bash
```

This should build your new firmware and create two directories: ICED-L.gcc and ICED-R.gcc. Those contain the binary files to flash.

```bash
# Connect only your left board and enter flash mode
sudo dfu-util --download ICED-L.gcc/kiibohd.dfu.bin

# Connect only your right board and enter flash mode
sudo dfu-util --download ICED-R.gcc/kiibohd.dfu.bin
```

At this point, your Ergodox Infinity should be both flash with your layout and your custom logo. Happy hacking!
