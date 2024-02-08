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

### Prototyping

Most of the above module types have a prototype version *(i.e. prototype.py)* which is used to test PiFire on other platforms (i.e. Windows, Linux or MacOS).  Input and output is simulated, and can be used in conjunction with your own modules for testing purposes.  

When prototyping functionality it's helpful to run control.py in one process and the app.py in another process.  This way you can access the PiFire webui via your web browser of choice and monitor the console output of the control.py script in a separate window.  

### API

PiFire offers a basic API to adjust settings or to control the system over the a network.  This API is used by the dashboard javascript to control the grill, which can be used as a reference for others build apps, dashboards or even other hardware that interfaces with the system.  

There are two levels of API commands that are available.  High level API commands such as Get, Set and Cmd are provided as a very simple interface to interact with the PiFire system.  These have simple to access commands that can be executed with a URL directly.  Low level API commands are provided to allow power users to directly manipulate settings/configuration/control on the system either with specific changes or in bulk writes & reads.  

#### High Level API Commands

High Level API commands were introduced in v1.7.0 of PiFire.  High Level API commands can be executed with only the URL.  These commands can be called via POST or GET.  Commands will always return the basic structure in the format below: 

```json
{
  "data" : {},
  "message" : "Command was accepted successfully.", 
  "result" : "OK"
}
```

Any data returned will be in the "data" structure (described in commands below).  The message will provide any additional details about the command, especially errors if any occurred. The result will either be 'OK' or 'ERROR'.  

##### Get Commands 

###### Get Current Temp Data Structure 

```info
/api/get/current
```

Returns (Example): 
```json
{
"AUX": {},
"F": {
	"Probe1": 204,
	"Probe2": 206
},
"NT": {
	"Grill": 0,
	"Probe1": 0,
	"Probe2": 0
},
"P": {
	"Grill": 518
},
"PSP": 0,
"TS": 1707345482984
}
```

###### Get Temperature 

```info
/api/get/temp/{probe label}
```

Notes:  {probe_label} must be a valid probe label (not the actual probe name) which can be discovered by executing get current. 

Returns: 
```json
{ 
'temp' : <probe temperature> 
}
```

###### Get Current Mode 
```info
/api/get/mode
```

Returns (example): 
```json
{ 
"mode" : "Stop" 
}
```

###### Get Hopper Level 
```info
/api/get/hopper
```

Notes: Returns the hopper pellet level in percentage.  

Returns (example): 
```json
{ 
"hopper" : 75 
}
```

###### Get Timer Data
```info
/api/get/timer
```

Returns (example):
```json
{ 
"start" : 0, 
"paused" : 0,
"end" : 0, 
"shutdown" : false,
"keep_warm" : false,
}
```

###### Get Notify Data
```info
/api/get/notify
```

Returns (example abbreviated):
```json
[
	{
	"eta": null,
	"keep_warm": false,
	"label": "Grill",
	"name": "GrillMain",
	"req": false,
	"shutdown": false,
	"target": 0,
	"type": "probe"
	},
	...
	{
	"keep_warm": false,
	"label": "Hopper",
	"last_check": 0,
	"req": true,
	"shutdown": false,
	"type": "hopper"
	}
]
```

###### Get Status Information for Key Items
```info
/api/get/status
```

Returns (Example):
```json
{
"display_mode": "Stop",
"lid_open_detected": false,
"lid_open_endtime": 0,
"mode": "Stop",
"name": "Development",
"outpins": {
	"auger": false,
	"fan": false,
	"igniter": false,
	"power": false
},
"p_mode": 0,
"prime_amount": 0,
"prime_duration": 0,
"s_plus": false,
"shutdown_duration": 10,
"start_duration": 30,
"start_time": 0,
"startup_timestamp": 0,
"status": "",
"ui_hash": 5734093427135650890,
"units": "F"
}
```

##### Set Commands 


###### Set Primary Setpoint 
```info
/api/set/psp/{integer/float temperature}
```

###### Set Mode
```info
/api/set/mode/{mode} 
/api/set/mode/prime/{prime amount in grams}[/{next mode}]
/api/set/mode/hold/{integer/float temperature}
```
Where mode/next mode = 'startup', 'smoke', 'shutdown', 'stop', 'reignite', 'monitor', 'error', 'manual'

###### Set PMode
```info			
/api/set/pmode/{pmode value} 
```
Note: PMode value is between 0-9 

###### Enable/Disable Smoke Plus 
```info
/api/set/splus/{true/false}
```
True = Smoke Plus Enabled, False = Smoke Plus Disabled

###### Set Notify Data/Settings
```url
/api/set/notify/{object}/req/{true/false} 
/api/set/notify/{object}/target/{value}
/api/set/notify/{object}/shutdown/{true/false}
/api/set/notify/{object}/keep_warm/{true/false} 
```
{object} = probe label, 'Timer', or 'Hopper' 
Note that target/{value} is not valid for Timer or Hopper

###### Enable/Disable PWM Control
```url
/api/set/pwm/{true/false} 
```
True = Enable PWM Control, False = Disable PWM Control 
PiFire must be configured for the PWM platform or this will do nothing 

###### Set Duty Cycle
```url
/api/set/duty_cycle/{percent} 
```
Must be a value in 0-100 percent

###### Tuning Mode Enable/Disable
```url
/api/set/tuning_mode/{true/false} 
```
True = Tuning mode enabled, False = Tuning mode disabled

###### Timer Control
```url
/api/set/timer/start/{seconds} 
/api/set/timer/pause 
/api/set/timer/stop
/api/set/timer/shutdown/{true/false}
/api/set/timer/keep_warm/{true/false}
```
Start will default to 60s unless a time is specified.  Timer duration should be in seconds.  

Shutdown True = Grill will shutdown when timer expires

Keep Warm True = Grill will go to Hold Mode (keep warm temperature specified in settings) when timer expires. 

###### Manual Control
```warning
System must already be in Manual mode (see set/mode command)
```
```url
/api/set/manual/power/{true/false}
/api/set/manual/igniter/{true/false}
/api/set/manual/fan/{true/false}
/api/set/manual/auger/{true/false}
/api/set/manual/pwm/{speed}
```
True = On, False = Off

##### CMD Commands

###### Restart Scripts/Server 
```url
/api/cmd/restart
```
Restarts the scripts/server without reboot. 

###### Reboot Server 
```url
/api/cmd/reboot
```
Reboots the server. 

###### Shutdown Server 
```url
/api/cmd/shutdown
```
Shutdown the server. 


#### Low Level API Commands

The Low Level API has changed since v1.5.0 to include more flexibility of probe definitions and types.  

For the low level commands, you must specify the correct request type (either GET or POST) for these commands.  

##### GET Low Level API Commands 

###### Settings
__GET__ 
```url
/api/settings
```
Dumps all of the current settings from settings.json in JSON format

###### Control
__GET__
```url
/api/control
```
Dumps all of the current control from control.json in JSON format

###### Hopper 
__GET__
```url
/api/hopper
```
Dumps data regarding the hopper level and what pellets are currently loaded.  For example:
  ```json 
  {
    "hopper_level":100,
    "hopper_pellets":"Generic Alder"
  }
  ``` 
###### Current 
__GET__
```url
/api/current
``` 
Dumps JSON of some key current activity, for ex:

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

##### POST Low Level API Commands

These commands all you to change some or all of the settings/control data directly.  

You do not have to replicate all of the data in these data structures when you POST, and only need to include the data that you are modifying.  __The exception to this rule is if you are modifying a LIST.__  In the case where you are modifying a list, you must read/modify/write the entire list.  Example may be in `control['notify_data']` which is a list, you may only want to edit a single entry.  You may need to read the entire list, modify the data, and then write the entire list back.

###### Settings 
__POST__ 
```url
/api/settings
``` 
Change any settings data from settings.json.  Post format identical to the format of the settings.json file.  For ex: 

```json
{ "globals" : { "grill_name" : "Smokey the Bear" } }
```

###### Control 
__POST__ 
```url
/api/control
``` 
Change any control data from control.json.  Post format identical to the format of the control data structure in redis.  For ex: 

```json
{ "updated" : true, "mode" : "Startup" } 
```

This code will change the mode to 'Startup' and by setting the 'updated' key to 'true', will tell the control script to check for the mode change.
