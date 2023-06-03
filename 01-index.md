---
title: "PiFire Documentation"
permalink: /
sort: 1
---
# ![PiFireIcon](/img/launcher-icon-1x.png) PiFire
## Raspberry Pi Based Smoker Grill Controller 

```danger
The creator of this project takes no responsibility for any damage that you may do to your personal property including modifications to your smoker grill if you choose to use this project.  The creator also takes no responsibility for any resulting harm or damages that may come from issues with the hardware or software design.*  ***This project is provided for educational purposes, and should be attempted only by individuals who wish to assume all risks involved.***
```

```note
This project is continuously evolving, and thus this readme will likely be improved over time, as I find the inspiration to make adjustments.  That being said, I'm sure there will be many errors that I have overlooked or sections that I haven't updated. This project is something I've done for both fun and for self-education.  If you decide to implement this project for yourself, and run into issues/challenges, feel free to submit an issue here on GitHub.  However, I would highly encourage you to dig in and debug the issue as much as you can on your own for the sake of growing your own knowledge.  Also, I have a very demanding day job, a family, and lots of barbeque to make - so please have patience with me.*
```

### Introduction
This project was inspired by user dborello and his excellent PiSmoker project ([http://engineeredmusings.com/pismoker/](http://engineeredmusings.com/pismoker/) and [https://github.com/DBorello/PiSmoker](https://github.com/DBorello/PiSmoker)).  I encourage you to check it out and get a rough idea of how this all works.  This particular project was built around a Traeger Texas smoker grill platform, but should work for most older Traeger models (or other brands with similar parts like the older Pit Boss) built with similar parts (fan, auger, and igniter).  I've built the code in a way to be somewhat modular & extensible such that you can replace the grill platform with your own specific platform instead.  Newer Traeger grills with their newer wifi enabled controllers have DC components (instead of the AC Fan / Auger) and aren't covered by this project.  

Just as with the PiSmoker project, I had a few goals in mind.  I also wanted to have tighter temperature controls, wireless control, and plotting of the grill / meat temperatures.  In addition, I wanted to design this project such that original smoker controller could be used if needed.  This way, if I wanted to, I could use my controller as a monitoring device for temperatures instead of controlling the temperature and leave that up to the original controller.  Basically, it was my fallback plan in case my project didn't work out, or if I wanted to do a quick cook on the Traeger without using the fancy GUI.  

```note
**UPDATE (11/2021)** After spending lots of time with PiFire, I've finally removed the original controller in favor of using PiFire exclusivesly.  However, this doesn't mean that you can't still retain your original controller if you want.  Both modes work fine. 
```

I made some other choices that diverged from the PiSmoker project as well.  

1. I've decided to use a Raspberry Pi Zero W, to both improve the compact nature of the controller and to provide in-built WiFi support.  It seems that that Pi Zero has enough horsepower to do what we need here.  And it's small enough to fit inside most small project boxes.
2. I'm using Flask to provide a WebUI framework for the application.  Firebase was a good solution at the time, but given that it's not longer usable, I've created the web based interface to be used from a mobile device, tablet or computer.  
3. Instead of using the MAX31865 RTD ADC, I'm using an off the shelf ADC solution based on the ADS1115 device which seems to be readily available on sites like Amazon and Adafruit.  It also supports I2C, which makes the hardware design easier in my humble opinion. This does make the code a little more fiddley, because we will need to find and use coefficients for each probe type and the Steinhart-Hart formula to calculate temperatures reasonably accurately. Many thanks to the HeaterMeter project for providing inspiration on both the hardware and software implementation. [https://github.com/CapnBry/HeaterMeter](https://github.com/CapnBry/HeaterMeter)
4. Instead of using the LCD Display from Adafruit, I'm using a cheap and ubiquitous OLED device (SSD1306) that I happened to have on hand already.  It also supports I2C, so again less hardware design necessary to get things designed into the project.  It also can be supported via either the Adafruit Python libraries or the Luma Python libraries (which I used for this project).  This display is frankly very small and someday, I may consider upgrading to something a bit larger, but for now, I'm reasonably happy.  
5. As mentioned above, since this thing sits along side the existing controller, I designed this to use more relays to allow for selecting between the two.  This certainly can be modified to ignore the existing controller altogether, but I didn't want to be without my grill while I fiddled with this project.
6. Another software choice was to modify the OS install such that the /tmp folder is RAM.  This way, I can store much of the history and control data in memory instead of writing to the flash device constantly and causing it to wear out.  Luckily, this is a pretty simple prospect with Raspberry Pi OS and is handled in the setup below.  

What I did keep from dborello's project was the PID controller which was the heart of the project.  This file is largely untouched from his project intentionally so that I could retain the goodness and legacy of his great work.  This project itself would not be possible without his path-finding and generous sharing of the knowledge.

### Features

* WiFi Enabled Access & Control (WebUI) via computer, phone or tablet
* Multiple Cook Modes
	* _Startup Mode_ (fixed auger on times with igniter on)
	* _Smoke Mode_ (fixed auger on times)
	* _Hold Mode_ (variable auger on times) using PID for higher accuracy
	* _Shutdown Mode_ (auger off, fan on) to burn off pellets after cook is completed
	* _Monitor Mode_ - See temperatures of grill / probes and get notifications if using another controller or if just checking the temperatures any time.  
	* _Manual Mode_ - Control fan, auger and igniter manually.  
	* _Prime_ - Allows you to prime the firepot with pellets prior to a cook.  
* Supports several different OLED and LCD screens
	* SSD1306 OLED Display
	* ST7789 TFT Display
	* ILI9341 TFT Display (now with more rotation options)
* Physical Button Input / Control (depending on the display, three button inputs)
* Encoder support for, so you can control your grill with a spinny knob.
* One (1) Grill Probe and Many Food Probes
	* Tunable probe inputs to allow for many different probe manufacturers
	* Supports the ADS1115 ADC, ADS1015 ADC, and MAX31865 RTD devices for measuring probes
	* Probe tuning tool to help develop probe profiles
	* NEW - Any number of probe inputs, limited only by the number of devices that the Raspberry Pi can support
	* Virtual Probes to allow you to do things like averaging probes, finding highest and lowest values of certain probes, etc.  
* Cook Timer - Moved to the Top Bar for Easy Access
* Notifications (Grill / Food Probes / Timer)
	* Supports Apprise, IFTTT, Pushover, and Pushbullet Notification Services
* Smoke Plus Feature to deliver more smoke during Smoke / Hold modes
* Safety settings to prevent over-temp, startup failure, or firepot flameout (and overload)
* Save temperature history for all probes / set points to a cook file that can be updated with images, notes, and even downloaded to your devices.
* Wood Pellet Tracking Manager - Now includes estimates of pellet usage.
* Pellet Level Sensor Support
	* VL53L0X Time of Flight Sensor
	* HCSR04 Ultrasonic Sensor
* Socket IO for Android Application Support _(GitHub User [@weberbox](https://github.com/weberbox) has made a Android client app under development here: [https://github.com/weberbox/PiFire-Android](https://github.com/weberbox/PiFire-Android))_
* NEW! Recipes / Recipe Mode - Integrated recipe creation and a new mode for developing a recipe 'program' that will control the grill for you and follow the recipe that was programmed.  
* NEW! Updated and re-written dashboards.  The dashboard has been refreshed with a new look.  New 'basic' and 'default' dashboards. If you are technically inclined, you can even develop your own dashboard.  
* NEW! Control panel everywhere! Allow your control panel to be displayed on almost all pages in PiFire.  
* ...And much more!  

### Screenshots & Videos

The dashboard is where most of your key information and controls are at.  This is the screen that greets you when you access the PiFire WebUI on your computer, smart phone or tablet in a browser.

![Dashboard](img/webui/PiFire-Dashboard-00.png)

For those of us who like to see the data, PiFire allows you to graph and save your cook history.  It's also a great way to monitor your cook in realtime.  

![History](img/webui/PiFire-History-00.png)

PiFire also provides an optional Pellet Manager which can help you track your pellet usage, store ratings, check your pellet level if you have a pellet sensor equipped.  

![Pellet Manager](img/webui/PiFire-PelletManager-00.png)

This is what PiFire looks like on your mobile device.  And in these screen shots you'll notice that we have dark mode enabled.  This helps for viewing at night, or just if you like the dark theme better.  Personally I think it looks pretty slick.  

![Mobile Dashboard](img/webui/PiFire-Mobile-00.jpg)

![Mobile History](img/webui/PiFire-Mobile-01.jpg)

Below is an example comparison that I did on a real cook of the Traeger controller attempting to hold 275F and the PiFire holding at the same temperature.  The difference is very impressive.  The Traeger swings massively up to 25F over and under the set temperature.  However the PID in from PiSmoker does a great job holding roughly +-7 degrees.  And this is without any extra tuning. 

![Performance](img/photos/SW-Performance.jpg)

Here's a brief YouTube video giving a basic overview of the PiFire web interfaces.

### PiFire Overview Video

I recommend at least taking a peek at the PiFire overview video below.  It covers the basics of operation, settings and control.  

[![YouTube Demo](docs/photos/Video-Link-Image-sm.png)](https://youtu.be/zifl0_sfFBA)

[Link to our channel on YouTube](https://www.youtube.com/channel/UCYYs50U5QvHHhogx_rqs0Yg)

Here is a the latest version 2.0 of the hardware w/TFT screen and hardware buttons in a custom 3D printed enclosure. We've come a long way since v1.0.

![Hardware v2](img/photos/HW-V2-Display.jpg)

And if you're interested in seeing more builds from other users, we have a discussions thread [here](https://github.com/nebhead/PiFire/discussions/28) where others have posted pictures of their unique builds.  

### Discord
I've added a discord server [here](https://discord.gg/F9mbCrbrZS) which can be a great resource for all who want to get more information, want to share their own builds, or just chat about pellet cooking.  Looking forward to seeing you there. 

### Credits

Web Application created by Ben Parmeter, copyright 2020-2023. Check out my other projects on [github](https://github.com/nebhead). If you enjoy this software and feel the need to donate a cup of coffee, a frosty beer or a bottle of wine to the developer you can click [here](https://paypal.me/benparmeter).

Of course, none of this project would be available without the wonderful and amazing folks below.  If I forgot anyone please don't hesitate to let me know.  

* **PiSmoker** - The project that served as the inspiration for this project and where the PID controller is wholesale borrowed from.  Special mention to Dan for providing encouraging feedback from day one of this project.  Many thanks!  Copyright Dan Borello. [engineeredmusings.com](http://engineeredmusings.com/pismoker/) [github](https://github.com/DBorello/PiSmoker)

* **Circliful** - Beautiful Circle Gauges from the original PiFire.  This script is no longer utilized, but deserves continued recognition.  Extra special mention for Patric for providing great support to me via GitHub.  Copyright Patric Gutersohn & other contributors. [gutersohn.com](http://gutersohn.com/) [github](https://github.com/pguso/js-plugin-circliful)

* **SVG Gauge** - Credits to Aniket Naik for the excellent SVG-Gauge. Released under the MIT License Copyright (c) 2016 [(github profile)](https://github.com/naikus) [github.com](https://github.com/naikus/svg-gauge)

* **Bootstrap Color Picker** - Color picker for configuring the chart line colors. Copyright (c) 2017 Javi Aguilar. Released under MIT license. [github](https://github.com/itsjavi/bootstrap-colorpicker)

* **Bootstrap** - WebUI Based on Bootstrap 4.  Bootstrap is released under the MIT license and is copyright 2018 Twitter. [getbootstrap.com](http://getbootstrap.com)

* **JQuery** - Required by Bootstrap. Copyright JS Foundation and other contributors. Released under MIT license. [jquery.org/license](https://jquery.org/license/)

* **Popper** - Required by Bootstrap. Copyright 2016, 2018 FEDERICO ZIVOLO & CONTRIBUTORS. Released under MIT license. [popper.js.org](https://popper.js.org/)

* **Chartjs** - For the fancy charts. Copyright 2018 Chart.js Contributors. Released under MIT license. [chartjs.org](https://chartjs.org/)

* **FontAwesome** - Amazing FREE Icons that I use throughout this project.  Copyright Font Awesome.  Released under the Font Awesome Free License. [fontawesome.com](https://fontawesome.com/) [github.com](https://github.com/FortAwesome/Font-Awesome)

* **Luma OLED** - The OLED display module for Python that I use.  This is not distributed in this project, but deserves a shout-out.  Copyright 2014-2020 Richard Hull and contributors. Released under MIT License. [readthedocs.io](https://luma-oled.readthedocs.io/en/latest/) [github.com](https://github.com/rm-hull/luma.oled)

* **ADS1115 Python Module** - Python module to support the ADS1115 16-Bit ADC. Also not actually distributed with this project, but also deserveds recognition.  Copyright David H Hagan. [pypi.com](https://pypi.org/project/ADS1115/) [github.com](https://github.com/vincentrou/ads1115_lib)

* **Other Adafruit Modules** - Multiple Adafruit modules were also leveraged in the making of this project and deserve recognition, even if they aren't distributed in the project.  

* **Contributions from the Community** - Thank you to those of you who have rolled up your sleeves, built out this project and contributed back. Whether that be with contributions to the code, designing new hardware, ideas and suggestions, or with a coffee that you bought me along the way.  Thank you very much, you keep this project running!

### Licensing

This project is licensed under the MIT license.

```
MIT License

Copyright (c) 2020 - 2023 Ben Parmeter and Contributors

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
