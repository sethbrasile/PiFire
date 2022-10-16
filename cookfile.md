---
title: "Cookfile Format"
permalink: /cookfile
sort: 8
---
### Cookfile Format v1.0.1

Introduced with v1.3.5 of PiFire, the cookfile format saves information about a cook from start to finish.  The file includes metadata, history data, labels, events, comments, and assets (images).  This page will provide the technical details, such that others can utilize the same formatting in their software.  I'm providing this specification and making it freely available so that anyone can adopt it for their project or production product.  

```danger
This specification, and contents are currently under development and may be updated periodically to improve accuracy and information.  
```

#### Compression

PiFire uses the standard pyzipfile module to compress and decompress files with the '.pifire' extension.  PiFire uses the standard ZIP compression method (using zip_deflated option in the python module).  

#### Structure 
In broad strokes, the cookfile is a compressed zip formatted folder of files.  The structure of the file looks like this:

```
/
+ metadata.json 
+ events.json
+ graph-data.json
+ graph-labels.json
+ comments.json
+ assets.json

/assets
    + asset_01.jpg
    + asset_02.jpg
    ...
    + asset_nn.jpg

    /thumbs
        + asset_01.jpg
        + asset_02.jpg
        ...
        + asset_nn.jpg
```

The assets listed in the asset folder and thumbs folder are examples only and represent image files that may be present in the cookfile.  

#### Metadata

The metadata file stores the information about the file and general information about the cook. The format is as follows:

```json
{
  "endtime": 1665881268061,
  "id": "298e46ec-4cec-11ed-a4e6-b827eb4fc611",
  "starttime": 1665861135559,
  "thumbnail": "4a0c0a1d-4d01-11ed-9781-b827eb4fc60e.jpg",
  "title": "Beef Ribs",
  "units": "F",
  "version": "1.0.1"
}
```

* __endtime__: Time the cook ended.  (unix time) [Required]
* __id__: This is a randomly generated UUID for the file.  Some effort should be taken to ensure this is a unique ID so that it does not conflict with other files on the same system or other systems.  [Required]
* __starttime__: Time the cook started.  (unix time)  [Required]
* __thumbnail__: Filename of the thumbnail image for the cookfile.  Used to display on the file list and in the cookfile viewer.  If a filename is present, it should exist in the /assets/thumbnails/ folder.  Thumbnail may be empty but tag must be present. [Optional]
* __title__: Title/Name for the cookfile.  Default will match the filename which is in the format "YYYY-MM-DD--HHMM-Cookfile". Title can be empty but tag must be present. [Optional]
* __units__: Temperature units in either C (Celcius) or F (Farenheit). [Required] 
* __version__ : The version level of the cookfile.  This is intended to allow for backward compatibility with older cook file formats in the future. [Required]  

#### Events

The events file contains details for each of the stages of a particular cook, from startup to shutdown/stop.  The following is an example. 

```json
[
  {
    "auger_cycle_time": 15,
    "augerontime": 75.83552360534668,
    "augerontime_c": "75 s",
    "endtime": 1665861489.0545118,
    "endtime_c": "22:44:21",
    "estusage_i": "0.05 pounds (0.78 ounces)",
    "estusage_m": "22 grams",
    "fanontime": 0,
    "fanontime_c": 0,
    "grill_settemp": 0,
    "id": "437d430c-4cbd-11ed-b5ff-b827eb4fc63f",
    "mode": "Startup",
    "p_mode": 1,
    "pellet_brand_type": "Bear Mountain Gourmet",
    "pellet_level_end": 36,
    "pellet_level_start": 36,
    "smart_start_profile": 1,
    "smokeplus": true,
    "starttime": 1665861128316.4575,
    "starttime_c": "12:12:08",
    "startup_temp": 70,
    "timeinmode": "-1664195266 s"
  },
  {
    "auger_cycle_time": 0,
    "augerontime": 2030.603833913803,
    "augerontime_c": "2030 s",
    "endtime": 1665873322.951159,
    "endtime_c": "22:44:33",
    "estusage_i": "1.34 pounds (21.48 ounces)",
    "estusage_m": "609 grams",
    "fanontime": 0,
    "fanontime_c": 0,
    "grill_settemp": 185,
    "id": "1aa09d90-4cbe-11ed-9770-b827eb4fc620",
    "mode": "Hold",
    "p_mode": 0,
    "pellet_brand_type": "Bear Mountain Gourmet",
    "pellet_level_end": 4,
    "pellet_level_start": 36,
    "smart_start_profile": 0,
    "smokeplus": true,
    "starttime": 1665861489258.294,
    "starttime_c": "12:18:09",
    "startup_temp": 0,
    "timeinmode": "-1664195615 s"
  },
  {
    "auger_cycle_time": 0,
    "augerontime": 0,
    "augerontime_c": "0 s",
    "endtime": 0,
    "endtime_c": 0,
    "estusage_i": "0.0 pounds (0.0 ounces)",
    "estusage_m": "0 grams",
    "fanontime": 0,
    "fanontime_c": 0,
    "grill_settemp": 0,
    "id": "286838fe-4cec-11ed-8e50-b827eb4fc646",
    "mode": "Stop",
    "p_mode": 0,
    "pellet_brand_type": "",
    "pellet_level_end": 0,
    "pellet_level_start": 0,
    "smart_start_profile": 0,
    "smokeplus": true,
    "starttime": 1665881269226.3845,
    "starttime_c": "17:47:49",
    "startup_temp": 0,
    "timeinmode": "NA"
  }
]
```

The above file consists of an __ordered__ list of events in a JSON structure.  Each event is a 'mode' which would have some kind of metrics associated with it.  The events/modes should be in order, with Startup or Prime as the beginning and Stop being the last mode such as:

1. Startup
2. Smoke
3. Hold
4. Shutdown
5. Stop

Each of these modes would have a block with metrics like the above.  The metrics listed above are the ones that PiFire will keep during a cook, but more may be added in future versions. 

* **auger_cycle_time:** Integer indicating fixed auger cycle time in seconds.  This value may be zero in certain modes (i.e. Hold, Shutdown, etc.)
* **augerontime:** Float indicating the amount of time the auger was enabled/on in seconds.  This value may be zero in certain modes. 
* **augerontime_c:** String converted from the augerontime seconds value. So that this number can be displayed nicely in the WebUI. 
* **endtime:** Float timestamp for the event/mode endtime.
* **endtime_c:** String converted from endtime, displayed in "HH:MM:SS" format. 
* **estusage_i:** String of estimated pellet usage calculated using augerontime, in imperial format i.e. "1.34 pounds (21.48 ounces)". 
* **estusage_m:** String of estimated pellet usage calculated using augerontime, in metric format i.e. "609 grams".
* **fanontime:** Float of operational fan time in this mode.  Currently not implemented.  
* **fanontime_c:** String converted of operational fan time in this mode. Currently not implemented.   
* **grill_settemp:** Integer or Float of grill set temperature (if applicable) during this mode.  
* **id:** String unique id for this particular mode.  Used by the software if there are multiple repeated modes/events in the same sequence. 
* **mode:** String indicating the mode.  
* **p_mode:** Integer indicating p-mode (0-10), if applicable. 
* **pellet_brand_type:** String indicating the pellet brand & type.  (i.e. "Bear Mountain Gourmet")
* **pellet_level_end:** Integer indicating the pellet level, in percent, at the end of this mode.  
* **pellet_level_start:** Integer indicating the pellet level, in percent, at the start of this mode.  
* **smart_start_profile:** Integer indication the Smart Start Profile selected, if applicable, for this mode.  
* **smokeplus:** Boolean indicating whether Smoke Plus feature is enabled.  'true' for enabled, 'false' for disabled.  
* **starttime:** Float timestamp for the event/mode start time. 
* **starttime_c:** String converted from starttime, displayed in "HH:MM:SS" format. 
* **startup_temp:** Integer or Float that indicates the ambient startup temperature (used to select smart-start)
* **timeinmode:** String calculated time in mode using endtime - starttime. 

#### Graph Data and Labels

These files contain information to help reconstruct a graph of the cook, using data from each of the probes.  The graph_labels file provides a mapping of labels to the probe data. 

```json
{
  "grill1_label": "Grill",
  "probe1_label": "Probe 1",
  "probe2_label": "Probe 2"
}
```

Currently PiFire will recognize 'grill1_label', 'grill2_label', 'probe1_label' and 'probe2_label' in this file.  Users can then map a string to to these particular datasets.  For example, Probe 1 may be for a prime-rib roast, so the user may choose to change this to "Prime Rib".  

* **grill1_label:** String to map to this probe label.  
* **probe1_label:** String to map to this probe label.
* **probe2_label:** String to map to this probe label.

The graph_data file provides the data for each of the probes. The below is an abbreviated version of the file.  You will notice that there are a series of ordered lists with data for each of the probes and the timestamps. 

```json
{
  "grill1_setpoint": [
    0,
    0,
    ...
    350,
    350
  ],
  "grill1_temp": [
    70,
    70,
    ...
    177,
    176
  ],
  "probe1_setpoint": [
    0,
    0,
    ...
    0,
    0
  ],
  "probe1_temp": [
    0,
    0,
    ...
    0,
    0
  ],
  "probe2_setpoint": [
    0,
    0,
    ...
    0,
    0
  ],
  "probe2_temp": [
    0,
    0,
    ...
    0,
    0
  ],
  "time_labels": [
    1665861135559,
    1665861138875,
    ...
    1665881264526,
    1665881268061
  ]
}
```


#### Comments

The comments file is used to store comments and notes about the cook.  Examples of usages could be to store recipe information, specific details about cooking, how well it turned out, including images of the cook.  This file may contain an empty list if no comments are added. (i.e. '[]')  

```json
[
  {
    "assets": [
      "6b809ae7-4d01-11ed-b360-b827eb4fc638.jpg",
      "7ca54370-4d01-11ed-bded-b827eb4fc5f9.jpg"
    ],
    "date": "2022-10-15",
    "edited": "2022-10-15 20:25",
    "id": "c00ca782-4d01-11ed-80dc-b827eb4fc620",
    "text": "A little disappointing in the final product.  Decent flavor, minimal meat and extremely chewy membrane.  It all looked great, but the texture wasn't my favorite.  Probably won't be doing these again.  ",
    "time": "20:22"
  },
  {
    "assets": [],
    "date": "2022-10-15",
    "edited": "",
    "id": "24ac3854-4d02-11ed-9420-b827eb4fc613",
    "text": "1. Started with a long smoke at 185F (spraying occasionally with a mixture of apple cider vinegar, soy sauce, Tapatio sauce)\n2. After internal temperature reached approximately 160F, wrapped meat side down (with sauce) and bumped temperature to 250F\n3. Finished last half an hour at 350F",
    "time": "20:25"
  }
]
```

* **assets:** List of strings of filenames, of the images that are to be displayed with this comment.  These images and their corresponding thumbnail images must be in the '/assets' and '/assets/thumbnails' folders.  
* **date:** String of the date that the comment was first entered. Format "YYYY-MM-DD".
* **edited:** String of the date that the comment was edited last in the format "YYYY-MM-DD HH:MM". 
* **id:** Unique ID for the comment. 
* **text:** Raw string of the comment itself.  The string can include line breaks with the '\n' null character.
* **time:** String of the time the comment was first entered.  Format "HH:MM".  

#### Assets

The assets file stores pointers to all of the asset (image) files in the assets folder.  If there are no assets in this cookfile, the list may be empty (i.e. '[]').  Here is an example of the format.  

```json
[
  {
    "filename": "4a0c0a1d-4d01-11ed-9781-b827eb4fc60e.jpg",
    "id": "4a0c0a1d-4d01-11ed-9781-b827eb4fc60e",
    "type": "jpg"
  },
  {
    "filename": "6b809ae7-4d01-11ed-b360-b827eb4fc638.jpg",
    "id": "6b809ae7-4d01-11ed-b360-b827eb4fc638",
    "type": "jpg"
  },
  {
    "filename": "7ca54370-4d01-11ed-bded-b827eb4fc5f9.jpg",
    "id": "7ca54370-4d01-11ed-bded-b827eb4fc5f9",
    "type": "jpg"
  }
]
```

* **filename:** String of the filename for the asset/media that is stored in the '/assets' folder.  [Required]
* **id:** String of the unique id assigned to this asset.  [Required]
* **type:** String of the filetype for this asset.  [Required]

Others notes about assets/media stored in the file.  

```note
Image files added to the cookfile are rotated, resized (800x600 maximum size) and converted to JPEG format by PiFire automatically.  When images are added to PiFire, they will automatically have a 128x128 thumbnail created, and added to the '/assets/thumbnails/' folder with a corresponding name.
```

#### Licensing

This cookfile format is licensed under the MIT license and is freely available to use in any project, free of charge. 

```
MIT License

Copyright (c) 2020 - 2022 Ben Parmeter and Contributors

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
```