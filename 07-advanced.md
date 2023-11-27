---
title: "Advanced Information"
permalink: /advanced
sort: 7
---
### Software Architecture

For the more adventurous of you, who want to tweak, extend and improve PiFire for your purposes, I've tried to architect PiFire in a way to make it friendly to use on different hardware.  

You may have noticed that we have a few modules that can be used interchangeably with a common API. Modules are organized into directories which are collections of modules that can be used for different purposes:

* **Platform (grillplat/xxxx.py)**: Used for Input / Output with grill itself.  (i.e. Any Inputs (Buttons / Switches) and Relay Outputs for Fan, Auger, Igniter, etc.)
* **Probes (probes/xxxx.py)**: Used to get data for the different temperature probes and devices.
* **Display (display/xxxx.py)**: Interface for the display / input.  Used to control and print information to the display.  
* **Distance / Hopper Level (distance/xxxx.py)**: Interface for getting hopper level input information - used by the pellet manager.
* **Controller (controller/xxxx.py)**: Controller logic, such as PID, fuzzy logic or machine learning, etc. 

Organizing modules in this way allows developers to create their own modules to support different ADC devices, Display Devices, and Grill configurations.  It's recommended to look at the existing modules to understand their construction.  Specifically looking at the prototype modules may be enlightening to get an idea of how they work.  

#### Prototyping

Most of the above module types have a prototype version *(i.e. prototype.py)* which is used to test PiFire on other platforms (i.e. Windows, Linux or MacOS).  Input and output is simulated, and can be used in conjunction with your own modules for testing purposes.  

When prototyping functionality it's helpful to run control.py in one process and the app.py in another process.  This way you can access the PiFire webui via your web browser of choice and monitor the console output of the control.py script in a separate window.  

#### API

PiFire offers a basic API to adjust settings or to control the system over the a network.  This API is used by the dashboard javascript to control the grill, which can be used as a reference for others build apps, dashboards or even other hardware that interfaces with the system.  

The API has changed since v1.5.0 to include more flexibility of probe definitions and types.  

- GET Settings (/api/settings): Dumps all of the current settings from settings.json in JSON format
- GET Control (/api/control): Dumps all of the current control from control.json in JSON format
- GET Hopper (/api/hopper): Dumps data regarding the hopper level and what pellets are currently loaded.  For example:
  ```json 
  {
    "hopper_level":100,
    "hopper_pellets":"Generic Alder"
  }
  ``` 
- GET Current (/api/current): Dumps JSON of some key current activity, for ex:

```json 
{
  "current":
  {
    "F": {
        "Probe1":114,
        "Probe2":90,
        "Probe3":151
    },
    "NT": {
        "Grill":0,
        "Probe1":0,
        "Probe2":0,
        "Probe3":0
    },
    "P": {
        "Grill":227
      },
    "PSP":0
  },
  "notify_data": [
    {
      "keep_warm":false,
      "label":"Grill",
      "name":"Grill",
      "req":false,
      "shutdown":false,
      "target":0,
      "type":"probe"
    },
    {
      "keep_warm":false,
      "label":"Probe1",
      "name":"Probe-1",
      "req":false,
      "shutdown":false,
      "target":0,
      "type":"probe"
    },
    {
      "keep_warm":false,
      "label":"Probe2",
      "name":"Probe-2",
      "req":false,
      "shutdown":false,
      "target":0,
      "type":"probe"
    },
    {
      "keep_warm":false,
      "label":"Probe3",
      "name":"Probe-3",
      "req":false,
      "shutdown":false,
      "target":0,
      "type":"probe"
    },
    {
      "keep_warm":false,
      "label":"Timer",
      "req":false,
      "shutdown":false,
      "type":"timer"
    },
    {
      "keep_warm":false,
      "label":"Hopper",
      "last_check":0,
      "req":true,
      "shutdown":false,
      "type":"hopper"
    }
  ],
  "status": {
    "display_mode":"Startup",
    "lid_open_detected":false,
    "lid_open_endtime":0,
    "mode":"Startup",
    "name":"",
    "outpins": {
      "auger":false,
      "fan":true,
      "igniter":true,
      "power":true
    },
    "p_mode":5,
    "prime_amount":0,
    "prime_duration":0,
    "s_plus":false,
    "shutdown_duration":60,
    "start_duration":240,
    "start_time":1699310434.8917897,
    "status":"active",
    "ui_hash":1382036478,
    "units":"F"
  }
}
```

- POST Settings (/api/settings): Change any settings data from settings.json.  Post format identical to the format of the settings.json file.  For ex: 

```json
{ "globals" : { "grill_name" : "Smokey the Bear" } }
```

- POST Control (/api/control): Change any control data from control.json.  Post format identical to the format of the control data structure in redis.  For ex: 

```json
{ "updated" : true, "mode" : "Startup" } 
```

This code will change the mode to 'Startup' and by setting the 'updated' key to 'true', will tell the control script to check for the mode change.