---
title: "Advanced Information"
permalink: /advanced
sort: 7
---
### Software Architecture

For the more adventurous of you, who want to tweak, extend and improve PiFire for your purposes, I've tried to architect PiFire in a way to make it friendly to use on different hardware.  

You may have noticed that we have a few modules that can be used interchangeably with a common API.

* **Grill Platform (grillplat_xxxx.py)**: Used for Input / Output with grill itself.  (i.e. Any Inputs (Buttons / Switche) and Relay Outputs for Fan, Auger, Igniter)
* **ADC (adc_xxxx.py)**: Used to get data for the different temperature probes (Grill, Probe1, Probe2).
* **Display (display_xxxx.py)**: Interface for the display.  Used to control and print information to the display.  
* **Distance / Hopper Level (dist_xxxx.py)**: Interface for getting hopper level input information - used by the pellet manager.  

Adding this layer for these components allows you to create your own modules to support different ADC devices, Display Devices, and Grill configurations.

#### Prototyping

Each of the above modules has a prototype version *(i.e. grillplat_prototype.py)* which is used to test PiFire on other platforms (i.e. Windows, Linux or MacOS).  Input and output is simulated, and can be used in conjunction with your own modules for testing purposes.  To enable prototype mode, simply set `prototype_mode = True` at the top of `control.py`.  After this has been changed, the software can be run on nearly any platform (Linux, Windows, Mac, Raspberry Pi, etc.) as long as python3 and flask are installed.  

When prototyping, functionality, it's helpful to run control.py in one process and the app.py in another process.  This way you can access the PiFire webui via your web browser of choice and monitor the console output of the control.py script in a separate window.  

More to come in this section.