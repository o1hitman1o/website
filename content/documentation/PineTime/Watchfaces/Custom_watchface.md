---
title: "Custom watchface"
draft: false
menu:
  docs:
    title:
    parent: "PineTime/Watchfaces"
    identifier: "PineTime/Watchfaces/Custom_watchface"
    weight:
aliases:
  - /wiki/PineTime_Custom_Watchface_Tutorial
---

This is a tutorial to help new users create custom watchfaces based on the InfiniTime Firmware for PineTime made by user JF002, thanks to him for its development.

We will explain some of the things we went through while creating some custom Watchfaces, so consider this as a log of experiments of sorts..

So stay tuned to it as it will be dynamically updated.

## What you need to start

The entire building process will be done by GitHub, so all you need is a device which can give you a GitHub Web Client, a PC, or tablet to give you enough screen space to review your code and a steady internet connection.

Since the compiling and file management is done by GitHub online, you have nothing else to worry about other than working with the files that display the watch face.

So with those things settled, let’s start with the basics of a watchface.

Please remember that this is a public documentation, so you can make an account and help us improve this page.

Please make sure not to unilaterally remove info though, but offer an alternative. If it is indeed a better way, in time your alternative will grow into the main text, and the latter info will be pruned.

## Overview

The Firmware (also called InfiniTime) we will be working with is made with a programming language named C++.

Basic knowledge of C/{cpp} is required to understand the advanced watch faces as that requires more complex code, but you can still do a some cool things without much knowledge of {cpp} programming, just some small edits to existing programs.

InfiniTime uses the LVGL graphics library to provide users with a simple and clean UI without overpowering the Nordic nRF52832 microcontroller, which is the brain of the watch.

To get the watchface to work, there are these basic steps. We will go over each step separately, so don’t be daunted, all will become clear soon.

For now, we will modify the existing watchface, change the positioning of the text labels, add an icon to an existing watchface and later on, we will do a full watchface.

The main file we need to focus on is the _WatchFaceDigital.cpp_ file, which contains most of the data attributed to what we see on the watchface, including The time/day characters, the battery, and Bluetooth icons, and also pedometer count.

So, everything we will be doing in the basic modifications is purely messing with this single file.

## Labels

### What are labels?
Labels are considered as "elemental" parts that make up a screen’s Text-based UI by the LVGL library. Each label is also considered as an "object" or "obj" and can be manipulated. By changing the data attributed with the "obj", for example, position, internal data like the strings/text etc. we can change what the label shows and where it shows it.

### Modifying label data

let’s observe something small like the word "BPM" near the bottom of the watch face.

{{< figure src="/documentation/images/PineTimeCustom-1.png" width="200" >}}

If we take a look at the source file of the watchface, (a.k.a. the clock.cpp file) we can observe that these particular lines are what attributes to the word "BPM" being displayed.

```CPP
heartbeatBpm = lv_label_create(lv_scr_act(), NULL);
lv_label_set_text(heartbeatBpm, "BPM");
```

We can modify the text inside the quotes and replace the word 'BPM' between those quotes to something like 'LOVE' and the result after compiling the firmware again with the changes and flashing it to the watch would be that the text changes on the watch face and displays 'LOVE' in place of 'BPM'.

If this happened correctly, then you have successfully made a custom new watchface|Now we can do something a bit more complex.

Now take for in instance the days of the week that we have on the bottom line with the date.

they are stored a "C array" which is basically multiple words separated by commas..
the date is in a format of **<day of the week> <day> <Month> <year>**

which in the source file is expressed as,

```CPP
sprintf(dateStr, "%s %d %s %d", DayOfWeekToString(dayOfWeek), day, MonthToString(month), year);
lv_label_set_text(label_date, dateStr);
```

here,

```CPP
%s %d %s %d
```

stores the print format of the variables, and

```CPP
DayOfWeekToString(dayOfWeek), day, MonthToString(month), year
```

are the variables themselves

Now, if the date was a Saturday 11/7/2020, you can observe that the date looks like

```
SATURDAY 11 JUL 2020
```

as seen in the above image (the one where we changed BPM to LOVE)

by modifying the format of the variables, we can change how those words are arranged, and add some extra characters if we like (there is a catch to the list of characters you can use however, but it will be discussed later)

For example,

If we modify the format by adding a comma "," in between the second "%s" and "%d" like this,

```CPP
"%s %d %s, %d"
```

and changing that in the line

```CPP
sprintf(dateStr, "%s %d %s, %d", DayOfWeekToString(dayOfWeek), day, MonthToString(month), year);
```

we can make the date become

```
SATURDAY 11 JUL, 2020
```

which means we now were able to modify how the text got displayed to make it a bit more nice

but if you haven’t noticed, the line containing the date is already full, meaning we will get some problems while displaying the date and causing it to wrap around, making a single character go to the next line and look more like,

```
SATURDAY 11 JUL, 202
0
```

So, why don’t we shorten the characters present in the date from being "SATURDAY" to simply just "sat." (It will have the small period at the end, and is only 3 characters long). I will also convert the months of the year from Capital to small letters.

For that look into the part where the days of the week of are stored as text,
and also while looking at it, we can solve another question, why was there two variables in the date format that looked like, DayOfWeekToString(dayOfWeek), and MonthToString(month) ?

It is because the system gives the date/ time as numbers (Monday-1, Tuesday-2 Wednesday-3 for the days, and 1-January, 2-February, 3-March ),
and so, a function along with a C array is used to assign these numbers to Days/Months in text form as it is easier to read.

this is the Array containing the day of the week, (as text)

```CPP
char const *Clock::DaysString[] = {
        "",
        "MONDAY",
        "TUESDAY",
        "WEDNESDAY",
        "THURSDAY",
        "FRIDAY",
        "SATURDAY",
        "SUNDAY"
};
```

and this Array stores the months of the year, (as text)

```CPP
char const *Clock::MonthsString[] = {
        "",
        "JAN",
        "FEB",
        "MAR",
        "APR",
        "MAY",
        "JUN",
        "JUL",
        "AUG",
        "SEP",
        "OCT",
        "NOV",
        "DEC"
};
```

Here we can see that the days are stored in a full format as "SUNDAY", "MONDAY", "TUESDAY" etc. We can change all of them to a shorter format like "sun.", "mon.", "tue.", to make it short and nice. While doing so, we can even make the months use small letters, as said before.

so the source file (clock.cpp) becomes,

(for the days of the week)

```CPP
char const *Clock::DaysString[] = {
        "",
        "mon.",
        "tue.",
        "wed.",
        "thu.",
        "fri.",
        "sat.",
        "sun."
};
```

and

(for the months of the year)

```CPP
char const *Clock::MonthsString[] = {
        "",
        "jan",
        "feb",
        "mar",
        "apr",
        "may",
        "jun",
        "jul",
        "aug",
        "sep",
        "oct",
        "nov",
        "dec"
};
```

which means now our original date, Saturday 11/7/2020 will become

```
sat. 11 Jul, 2020
```

you now know how to change the data present in a label object, and the format of it..,

Here is a fun idea you can try: you can even replace the days with whatever thing that tells you (or) reminds you the day of the week
(like the food served in the café, Monday/taco, Tuesday/burger, Wednesday/pasta etc.)

{{< admonition type="note" >}}
 When making the custom array, don’t forget to leave an empty "" as the first element of the array, This is because the date is given by the system in a natural numbers format (1,2,3, and so on) rather than a zero-starting format (0,1,2,3, and so on), which the C array uses to index. So the C array indexes the days as ""-0, "Monday"-1, "Tuesday"-2 etc. and the months as ""-0, "January"-1, "February"-2 and so on.
{{</ admonition >}}

### Label positioning

The locational placement in LVGL is done on a Cartesian plane, where each object can have dynamic origin placement, and the Y-axis is inverted. So going down is done with a positive Y-axis value and not negative as it is by default.

{{< figure src="/documentation/images/LVGL_coord_system.png" caption="LVGL coord system" width="200" >}}

The position of the various objects in WatchFaceDigital.cpp are set by the line,

```CPP
lv_obj_set_pos(<obj>, <new_x>, <new_y>)
```

and the top-left corner is the Cartesian origin, aka coordinates (0,0)

this image can show you how to decide label placement for lv_obj_set_pos(...)

We use another function, that is more advanced, that gives the positional alignment based on preset locations

```CPP
lv_obj_align(obj, obj_ref, LV_ALIGN_..., x_ofs, y_ofs);
```

**obj** is your text label

**obj_ref** is a reference object to which obj will be aligned.
If obj_ref = NULL , then the parent of obj will be used.
If obj_ref = lv_scr_act(), then the whole screen will be used.

**LV_ALIGN_...** is the type of alignment; inside another object or next to the reference, for example IN_TOP_LEFT, OUT_BOTTOM_MID, ...

**x_ofs, y_ofs** allow you to shift the object by a specified number of pixels after aligning it

Label positioning based on alignment is both a simple and complicated thing to understand, so here I have given something you can refer to while modifying the position of the various labels and objects.

You can also refer here to LVGL’s documentation of coordinate system https://docs.lvgl.io/master/overview/coords.html

List of the possible alignments: https://docs.lvgl.io/latest/en/html/widgets/obj.html#alignment

It is however recommended that you use the first method to set the location

```CPP
lv_obj_set_pos(<obj>, <new_x>, <new_y>)
```

as it is simple and easier for beginners

Here is a small example.

Take the Label that tells the date,
In the Digital Clock source file (WatchFaceDigital.cpp) it is this line,

```CPP
lv_obj_align(label_date, lv_scr_act(), LV_ALIGN_CENTER, 0, 60);
```

by increasing the Value of the Y coordinate (60) to a higher value, we can bring the position of the Date downwards a bit away from the Time, and toward the Heartbeat count in the bottom row
here I will increase it to 80, so it becomes..

```CPP
lv_obj_align(label_date, lv_scr_act(), LV_ALIGN_CENTER, 0, 80);
```

and now we have made some space up top..

now let’s try something a bit complex,

Take the position argument for the label that tells you time. Here, in the source file (WatchFaceDigital.cpp),

```CPP
lv_obj_align(label_time, lv_scr_act(), LV_ALIGN_IN_RIGHT_MID, 0, 0);
```

this line determines the position of the Label telling time, as seen in the image

we’re modifying this, by changing the origin alignment parameter (here it is LV_ALIGN_IN_RIGHT_MID) to LV_ALIGN_IN_TOP_LEFT

you can alternatively swap the whole line to:

```CPP
lv_obj_set_pos(label_time, 0, 0);
```

this makes the Time label/obj. to go to the top-left corner

but I will do something a little extra,
I will modify the label that store the data and Time format,
i.e this line,

```CPP
sprintf(timeStr, "%c%c:%c%c", hoursChar[0],hoursChar[1],minutesChar[0], minutesChar[1]);
```

by removing the ":" colon in between the numbers, and replacing it with a Newline symbol "\n"
I change it to become,

```CPP
sprintf(timeStr, "%c%c\n%c%c", hoursChar[0],hoursChar[1],minutesChar[0], minutesChar[1]);
```

this gives it a nice wrapped text format in the top corner, and gives us some space to play with in the side, for things like Pictures and icons, which we will do next..

If you have been able to do these things, you now have completed the 2nd part of the tutorial, and now know how to change and modify the position of labels.

## Using icons

The LVGL library allows for the use of widgets known as "Images", In short it allows you to use small Icons like pictures with a small dedicated function, However, when this was attempted the first time we stumbled on some problems as LVGL v6 (used on the PineTime) is not much documented as the latest release (v7 as of August 2020) but also the existing code was only documented for C not {cpp}, after some painful attempts we were able to translate it into {cpp},

To bring images into Clock.cpp you will need to do the following,

1. Have a small image that cannot exceed a maximum size of 240px x 240px (PineTime max resolution)
2. Use this Image converter (Thanks to LVGL) https://lvgl.io/tools/imageconverter to convert your image to a C array and having the Color format as "True color" and the output format as "C array". Make sure to use something simple as the name we will be using "bitmap" as the name, but will also be referred as <name> for simplicity

{{< admonition type="note" >}}
 for example we shall use <name> = bitmap, but any simple word can be used, as long as it does not cause problems with system variables
{{</ admonition >}}

### Image size considerations

since the image will be using the flash directly, we need to be considerate about flash memory usage.

```
<picture_X> x <picture_Y> x 2
```

gives you the number of KB the image used in storage

where, <picture_X> <picture_Y> are the dimensions of the image horizontally and vertically

for example,

```CPP
if <picture_X>=80px <picture_Y>=64px
```

then,

```CPP
total storage used = 80 x 60 x 2 = 10.24KB
```

Please use the flash storage with consideration, when using other apps as well, excess usage of storage might mean the Firmware will not compile. The limit to storage to about 400Kb for the user, the firmware size must not exceed that.

## Preparing the image for inclusion as an icon

Once you have obtained your C array from the LVGL converter, you can take a look inside it to see all the different formats of your image, try using something like Notepad++ or any of your favorite text editors to peek inside it,

there will be 4 sets of Arrays inside it that look like,

```CPP
#if LV_COLOR_DEPTH == 1 || LV_COLOR_DEPTH == 8
 /*Pixel format: Red: 3 bit, Green: 3 bit, Blue: 2 bit*/
 0x00, 0x00, 0x00,...
...0x00, 0x00, 0x00, 
#endif

#if LV_COLOR_DEPTH == 16 && LV_COLOR_16_SWAP == 0
 /*Pixel format: Red: 5 bit, Green: 6 bit, Blue: 5 bit*/
 0x00, 0x00, 0x00,...
...0x00, 0x00, 0x00, 
#endif

#if LV_COLOR_DEPTH == 16 && LV_COLOR_16_SWAP != 0
 /*Pixel format: Red: 5 bit, Green: 6 bit, Blue: 5 bit BUT the 2 bytes are swapped*/
 0x00, 0x00, 0x00,...
...0x00, 0x00, 0x00, 
#endif

#if LV_COLOR_DEPTH == 32
 /*Pixel format: Fix 0xFF: 8 bit, Red: 8 bit, Green: 8 bit, Blue: 8 bit*/
 0x00, 0x00, 0x00,...
...0x00, 0x00, 0xff, 
#endif
};
```

And another small bit of info we will need for later that looks like,

```CPP
 const lv_img_dsc_t bitmap = {
  .header.always_zero = 0,
  .header.w = 40,
  .header.h = 40,
  .data_size = 1600 * LV_COLOR_SIZE / 8,
  .header.cf = LV_IMG_CF_TRUE_COLOR,
  .data = bitmap_map,
 };
```

{{< admonition type="note" >}}
 There are some header files at the top, which we can ignore.
{{</ admonition >}}

### RGB565 image format

The PineTime uses a display that uses a 16 bit color space, also known as RGB565.

These 16 bit are assigned to RGB as 5 bits each for Red and Blue and 6 bits for Green, so 5+6+5=16 bits are required, so each pixel’s color occupies 2 bytes of data,
and since 2^16^ is equal to 65,536 it allows us to view 65,536 or 65k colors

The way it packs these bits is by converting the bits into 2x  4+4 bit hex-code, so for example,

if the color of a pixel in Binary is **10110100 01011111** (this color is approximately Lavender purple)

It is split as **1011** & **0100** for the first byte and **0101** & **1111** for the second byte
and so, converting the binary into Hex-code,

the two parts are **0xB4** and **0xF5**

These two parts in conjunction are used for determining the color of one pixel.

also from the binary, it is observed that,

The bits **10110** is used for Red, **100010** is used for green, and **11111** is used for blue.

### Flipping the bytes

The LVGL library has a feature that allows you to flip the two bytes of the pixel, so if the two parts were, ...0xB4,0xF5,... ,it will change it to become, ...0xF5,0xB4,...

The reason for this is to allow the use of 8-bit SPI interfaces, but we do not require it, and if set with  wrong parameter we could get problems with the color...

To make sure you are ready for the next step, make sure that inside your LVGL configuration file (located at **src/libs/lv_conf.h**)

this parameter,

```CPP
#define LV_COLOR_16_SWAP   1
```

is set to "1" as seen.

{{< admonition type="note" >}}
 If you haven’t modified it or tampered with it with your GitHub fork, you shouldn’t have a problem as it is correct by default, and you can skip these steps
{{</ admonition >}}

### Creating an Object from the Array

To include the Icon, first Identify the Array you need to copy to the source (clock.cpp)

The one we require from it is the data below the tag that looks like:

```CPP
#if LV_COLOR_DEPTH == 16 && LV_COLOR_16_SWAP != 0
/*Pixel format: Red: 5 bit, Green: 6 bit, Blue: 5 bit BUT the 2 bytes are swapped*/
0x00, 0x00, 0x00,...
...0x00, 0x00, 0x00, 
#endif
```

from this copy the Data from the array alone...
I.e this part,

```CPP
0x00, 0x00, 0x00,...
...0x00, 0x00, 0x00
```

(Make sure to not include the comma at the end or the #endif as the entire part is going to substitute a new array)

In clock.cpp, just below the header files and the Task creation part (I.e event_handler...),

```CPP
static void event_handler(lv_obj_t * obj, lv_event_t event) {
Clock* screen = static_cast<Clock *>(obj->user_data);
screen->OnObjectEvent(obj, event);
}
```

create a name for the label with,

```CPP
static lv_img_dsc_t <name>; // remember to replace <name> with the actual name you gave to your image while converting!
```

then below it create a array to hold the data with,

```CPP
const uint8_t <name>_map[] = {}; // paste the array you copied from the conversion file we specified above...
```

so your  array is something like,

```CPP
const uint8_t <name>_map[] = {0x00,0x00,0x00...
...0x00,0x00,0x00};
```

so your Entire top region of declaration looks like,

```CPP
#include <cstdio>
#include <libs/date/includes/date/date.h>
...
using namespace Pinetime::Applications::Screens;
extern lv_font_t jetbrains_mono_extrabold_compressed;
extern lv_font_t jetbrains_mono_bold_20;
extern lv_style_t* LabelBigStyle;
   
static void event_handler(lv_obj_t * obj, lv_event_t event) {
 Clock* screen = static_cast<Clock *>(obj->user_data);
 screen->OnObjectEvent(obj, event);
}
   
//Declare the descriptor here
static lv_img_dsc_t <name>;
//place the Image data here
const uint8_t <name>_map[] = {0x00,0x00,0x00...
...0x00,0x00,0x00
};
```

{{< admonition type="note" >}}
 Declaring variables outside a function like we did above is known as global scope declaration, this means the variable can be used by not just one function but the Entire code.
{{</ admonition >}}

Then inside the

```CPP
Clock::Clock(DisplayApp* app,...){...
```

region (the watchface function), you need to place a particular set of lines which LVGL uses to define the object to declare the array as an Icon/Image, You can place this set of lines above _label_time_.

```CPP
<name>.header.always_zero = 0; //Initialization
<name>.header.w = <picture_X>;                     // Setting the Width (or) Horizontal length of the image (number of px)
<name>.header.h = <picture_Y>;                     // Setting the Height (or) vertical length of the image (number of px)
<name>.data_size = <Hr_length> * <Vr_length> * LV_COLOR_SIZE / 8; //Allocation of memory for the image
<name>.header.cf = LV_IMG_CF_TRUE_COLOR; // Sets the color scheme for the image
<name>.data = <name>_map;                // Maps the Image data to the Array
lv_obj_t *img_src = lv_img_create(lv_scr_act(), NULL);  // Create an image object
lv_img_set_src(img_src, &<name>);        // Set the created file as image (<name>)
```

again, make sure to replace <name> with the name you gave it during conversion!

Now that we have bought in the image data, we need to set the position, you can place this just below the lines we wrote for bringing in the image, It can be done with either,

```CPP
lv_obj_set_pos(img_src, <x_pos, <y_pos>); // <x_pos>, <y_pos> are the coordinates of the Cartesian plane
```

or,

```CPP
lv_obj_align(img_src, lv_scr_act(), LV_ALIGN_<parameter>, <x_pos, <y_pos>); 
```

If done correctly, you will now have a beautiful little Icon/Image in your Watch face, make sure that your Watch face can accommodate the image by pushing the other labels farther away, creating space for it.

We have provided a small template you can use for adding even a large image comfortably

If you have succeeded with this, you have completed part 3 of the tutorial.

## Creating an entirely new watchface

The instructions above describe how to modify the existing default watchface, if you would like to create a new watchface instead you will need to complete some additional steps. We will refer to the new watchface as WatchFaceName in these instructions.

### Create the watchface files

The watchface is composed of 2 files, WatchFaceName.cpp and WatchFaceName.h. You can copy them from one of the existing watchfaces and give it a new name to provide a basic layout to start from. It is important to increment the ClockFace number near the top of WatchFaceName.cpp otherwise the wrong watchface will be displayed when leaving the menu.

```CPP
settingsController.SetClockFace(0);
```

### Add the watchface to Clock.cpp and Clock.h

Clock.cpp now provides the ability to switch between multiple watchfaces by long-pressing the screen. You will need to make 3 modifications in Clock.cpp and 2 modifications in Clock.h.

**src/displayapp/screens/Clock.cpp**

```CPP
#include "WatchFaceDigital.h"
#include "WatchFaceAnalog.h"
#include "WatchFaceName.h"

               [this]() -> std::unique_ptr<Screen> { return WatchFaceDigitalScreen(); },
               [this]() -> std::unique_ptr<Screen> { return WatchFaceAnalogScreen(); },
               [this]() -> std::unique_ptr<Screen> { return WatchFaceNameScreen(); },

std::unique_ptr<Screen> Clock::WatchFaceAnalogScreen() {  
  return std::make_unique<Screens::WatchFaceAnalog>(app, dateTimeController, batteryController, bleController, notificatioManager, settingsController);
}

std::unique_ptr<Screen> Clock::WatchFaceNameScreen() {  
  return std::make_unique<Screens::WatchFaceName>(app, dateTimeController, batteryController, bleController, notificatioManager, settingsController, heartRateController);
}

```

**src/displayapp/screens/Clock.h**

```CPP
         ScreenList<3> screens;
         std::unique_ptr<Screen> WatchFaceDigitalScreen();
         std::unique_ptr<Screen> WatchFaceAnalogScreen();
         std::unique_ptr<Screen> WatchFaceNameScreen();
```

Be sure to increment the number of screens.

### Add the watchface to CMakeLists.txt

**src/CMakeLists.txt**

```CPP
       ## Watch faces
       displayapp/icons/bg_clock.c
       displayapp/screens/WatchFaceAnalog.cpp
       displayapp/screens/WatchFaceDigital.cpp
       displayapp/screens/WatchFaceName.cpp
```
## Creating an entirely new watchface (Updated Alternative)

The previous method may not work with the current version of Infinitime as of (2023 Jan 28). Therefore, here I will tell you a method of creating watch faces on the current build.

### Create the watch face files

The watch face is composed of 2 files, WatchFaceName.cpp and WatchFaceName.h. You can copy them from one of the existing watch faces and give it a new name to provide a basic layout to start from.

Important do not forget to rename the class names to reflect the new filenames.

### Add the watchface to Clock.cpp and Clock.h

Clock.cpp now provides the ability to switch between multiple watchfaces by long-pressing the screen. You will need to make 3 modifications in Clock.cpp, 1 modification in Clock.h and two modifications in SettingsWatchFace.h which will allow us to select the newly created watch face.

**src/displayapp/screens/Clock.cpp**

```CPP
#include "displayapp/screens/WatchFaceDigital.h"
#include "displayapp/screens/WatchFaceAnalog.h"
#include "displayapp/screens/WatchFaceName.h"



switch (settingsController.GetClockFace()) {
       case 0:
         return WatchFaceDigitalScreen();
         break;
       case 1:
         return WatchFaceAnalogScreen();
         break;
       case 2:
         return WatchFacePineTimeStyleScreen();
         break;
       case 3:
         return WatchFaceTerminalScreen();
         break;
       case 4:
         return WatchFaceInfineatScreen();
         break;
       case 5:
         return WatchFaceCasioStyleG7710();
         break;
       case 6:
         return WatchFaceNameScreen();
         break;
     }
     return WatchFaceDigitalScreen();
   }

std::unique_ptr<Screen> Clock::WatchFaceAnalogScreen() {  
  return std::make_unique<Screens::WatchFaceAnalog>(app, dateTimeController, batteryController, bleController, notificatioManager, settingsController);
}

std::unique_ptr<Screen> Clock::WatchFaceNameScreen() {  
  return std::make_unique<Screens::WatchFaceName>(app, dateTimeController, batteryController, bleController, notificatioManager, settingsController, heartRateController);
}
```

**src/displayapp/screens/Clock.h**

```CPP
       std::unique_ptr<Screen> screen;
       std::unique_ptr<Screen> WatchFaceDigitalScreen();
       std::unique_ptr<Screen> WatchFaceAnalogScreen();
       std::unique_ptr<Screen> WatchFaceNameScreen();
```

Since I have set WatchFaceName to case 6 in a switch statement beforehand it will take sixth position in a list.

**src/displayapp/screens/settings/SettingWatchFace.h**

```CPP
         #include "displayapp/screens/WatchFaceInfineat.h"
         #include "displayapp/screens/WatchFaceCasioStyleG7710.h"
         #include "displayapp/screens/WatchFaceName.h"

          std::array<Screens::CheckboxList::Item, settingsPerScreen * nScreens> watchfaces {
         {{"Digital face", true},
          {"Analog face", true},
          {"PineTimeStyle", true},
          {"Terminal", true},
          {"Infineat face", Applications::Screens::WatchFaceInfineat::IsAvailable(filesystem)},
          {"Casio G7710", Applications::Screens::WatchFaceCasioStyleG7710::IsAvailable(filesystem)},
          {"Name Face", true},
          {"", false}}};
       ScreenList<nScreens> screens;

```

### Add the watchface to CMakeLists.txt

**src/CMakeLists.txt**

```CPP
       ## Watch faces
       displayapp/icons/bg_clock.c
       displayapp/screens/WatchFaceAnalog.cpp
       displayapp/screens/WatchFaceDigital.cpp
       displayapp/screens/WatchFaceName.cpp
```

## Using git to work on the firmware

### Cloning the repository

Instructions for cloning the repository are available on the [Building and programming page](https://github.com/JF002/InfiniTime/blob/develop/doc/buildAndProgram.md) on github.

#### Changing the code to add the image

Use the editor of your choice to modify the source files. Please read the [coding conventions](https://github.com/InfiniTimeOrg/InfiniTime/blob/main/doc/coding-convention.md) before you start.

### Compiling the firmware

Information about how to compile the firmware is included on the [Building and programming page](https://github.com/JF002/InfiniTime/blob/develop/doc/buildAndProgram.md) on github.

### Testing the firmware

#### Installing the new firmware

A holistic guide on how to install different firmware using various hardware programmers is available here: [Reprogramming the PineTime](/documentation/PineTime/Software/Reprogramming/).

If you would like to install the firmware by OTA/DFU, you can follow these steps:

```console
cmake -DARM_NONE_EABI_TOOLCHAIN_PATH=/path/to/gcc-arm-none-eabi-9-2020-q2-update -DNRF5_SDK_PATH=/path/to/nRF5_SDK_15.3.0_59ac345 -DUSE_OPENOCD=1 -DBUILD_DFU=1 ../
make -j pinetime-mcuboot-app
```

Be aware the paths for the cmake command must be absolute. The -DBUILD_DFU argument will generate a zip file which can be flashed using nRF Connect ([not recommended](https://github.com/InfiniTimeOrg/InfiniTime#companion-apps)) or Gadgetbridge on Android. You must have adafruit-nrfutil installed in your $PATH for this to work.
