---
title: "Recipe File Format"
permalink: /recipefile
sort: 9
---
## Recipe File Format v1.0.0

Uses a very similar format to the cookfile format such that many of the same functions can be re-used to manage the file.  In fact, the cookfile functions that are shared, have been moved into new module files to both simplify the implementation and to share code.  The below is a quick review of the new files that have been created to support both the cook file and recipe file formats. 

+ __file_common.py__

  + __read_json_file_data(filename, jsonfile, unpackassets)__ - Read a JSON file out of the compressed file.  Also unpacks the assets when reading an asset JSON file, into a temporary folder.

  + __update_json_file_data(filedata, filename, jsonfile)__ - Writes a specific JSON file back to the compressed file.  Used to update specific JSON file in the compressed file structure.   Cannot be used to update assets (images) in the file. 

  + __fixup_assets(filename, jsondata)__ - This function is used to fix missing or incorrect assets in the compressed file.  Used if a file gets corrupted somehow.  

+ __file_cookfile.py__

  + __create_cookfile(historydata)__ - Creates a brand new cookfile and puts this into the history folder.  

  + __read_cookfile(filename)__- Reads all JSON files in the compressed file and places those into a larger JSON structure.  Also unpacks the assets into a temporary folder.

  + __upgrade_cookfile(cookfilename)__ - Updates the compressed file to the newer format.  

+ __file_recipes.py__

  + __create_recipefile()__ - Creates a brand new recipe file and puts this into the history folder.  

  + __read_recipefile(filename)__ - Reads all JSON files in the compressed file and places those into a larger JSON structure.  Also unpacks the assets into a temporary folder.

  + __convert_recipe_units(recipe, units)__ - Used to convert units in a recipe between imperial and metric.  

+ __file_media.py__

  + __add_asset(filename, assetpath, assetfile)__ - Add an asset (image) to a compressed file.  

  + __rotate_image(filepath, asset_id, filetype)__ - Rotate an image (if needed) using EXIF data of the image.  

  + __create_thumbnail(filepath, asset_id, filetype, crop)__ - Create a smaller thumbnail image from the uploaded full size image.  

  + __set_thumbnail(filename, thumbfilename)__ - Set a thumbnail image in metadata JSON of the compressed file. 

  + __unpack_thumb(thumbname, filename)__ - Function used to uncompress the thumbnail only for a specific compressed file into a temporary folder.  Used to display thumbs in a filelist without having to unpack ALL assets. 

```danger
This specification, and contents are currently under development and may be updated periodically to improve accuracy and information.  
```

### Compression

PiFire uses the standard pyzipfile module to compress and decompress files with the '.pfrecipe' extension.  PiFire uses the standard ZIP compression method (using zip_deflated option in the python module).  

### File Structure

In broad strokes, the recipe file is a compressed zip formatted folder of files.  The structure of the file looks like this:

```
/
metadata.json
recipe.json
comments.json
assets.json
    /assets
        image-00.jpg
        image-01.jpg
        ...
        image-99.jpg
        /thumbnails
            image-00.jpg
            image-01.jpg
            ...
            image-99.jpg
```

### Metadata

```json
{
  "author": "Ben Parmeter",
  "cook_time": 360,
  "description": "A simple baby-back ribs recipe that is often used as the gold standard for backyard BBQ ribs.",
  "difficulty": "Easy",
  "id": "8b9ee393-5c9d-11ed-b19d-edcb29010da6",
  "image": "",
  "prep_time": 15,
  "rating": 5,
  "thumbnail": "",
  "title": "3-2-1 Baby Back Ribs",
  "units": "F",
  "username": "nebhead",
  "version": "1.0.0",
  "food_probes": 2
}
```
* __author__ : (string) The name of the author of this recipe. [Optional]
* __cook_time__: (integer) This is the total estimated cook time for the recipe in minutes. 
* __description__:  (string) This is a text field, used to describe the recipe.  
* __difficulty__:  (string) This is a text field that describes the recipe difficulty level. (i.e. Easy, Intermediate, Hard, Advanced)
* __id__: (string) This is a randomly generated UUID for the file.  Some effort should be taken to ensure this is a unique ID so that it does not conflict with other files on the same system or other systems.  [Required]
* __image__: (string) Filename of the full size image/splash for the recipe.  If a filename is present, it should exist in the /assets/ folder.  Image may be empty but tag must be present. [Optional]
* __prep_time__: (integer) The estimated preperation time for this recipe in minutes.  
* __rating__:  (float) [0-5] Scale rating of the recipe. 
* __thumbnail__: (string) Filename of the thumbnail image for the recipe.  Used to display on the file list and in the recipe viewer.  If a filename is present, it should exist in the /assets/thumbnails/ folder.  Thumbnail may be empty but tag must be present. [Optional]
* __title__: (string) Title/Name for the recipe.  Default will match the filename which is in the format "YYYY-MM-DD--HHMM-Recipe". Title can be empty but tag must be present. [Optional]
* __units__: (char) Temperature units in either C (Celcius) or F (Farenheit). [Required] 
* __username__: (string) The username of the author for this recipe. [Optional]
* __version__ : (string) The version level of the cookfile.  This is intended to allow for backward compatibility with older file formats in the future. [Required]  
* __food_probes__: (integer) This is the number of food probes to be used in this recipe.  This number can be greater than zero even if the food probes aren't used.  

### Recipe

```json
{
  "ingredients": [
    {
        "name" : "Baby-back Ribs",
        "quantity" : "1 Rack",
        "assets" : []
    }, 
    {
        "name" : "BBQ Sauce",
        "quantity" : "1/2 Cup",
        "assets" : [] 
    }, 
    {
        "name" : "Seasoning/Rub",
        "quantity" : "4 Tbsp",
        "assets" : []
    }
  ],
  "instructions": [
    {
      "text" : "Prepare the ribs by removing the membrane on the back of the ribs with a knife, gently scoring along the bone.",
      "ingredients" : ["Baby-back Ribs"],
      "assets" : [],
      "step" : 0
    },
    {
      "text" : "Rub the ribs with binder (oil, mustard, water), then add your seasonsing / rub of choice.",
      "ingredients" : ["Seasoning/Rub"],
      "assets" : [],
      "step" : 0
    },
    {
      "text" : "Place the ribs on the grill and start the recipe program.",
      "ingredients" : [],
      "assets" : [],
      "step" : 0
    },
    {
      "text" : "After smoking for 3 hours, wrap the ribs in foil with meat side down, adding some water or apple juice to the wrap for moisture.",
      "ingredients" : [],
      "assets" : [],
      "step" : 2
    },
    {
      "text" : "After cooking for 2 hours wrapped, remove from foil wrap and place back on the grill.",
      "ingredients" : [],
      "assets" : [],
      "step" : 3
    },
    {
      "text" : "Finish by basting with barbeque sauce, and cook until the sauce firms up.",
      "ingredients" : [],
      "assets" : [],
      "step" : 3
    },
  ],
  "steps": [
    {
      "hold_temp": 0,
      "message": "",
      "mode": "Startup",
      "notify": false,
      "pause": false,
      "timer": 0,
      "trigger_temps": {
        "grill": 0,
        "probe1": 0,
        "probe2": 0
      }
    },
    {
      "hold_temp": 0,
      "message": "It's time to wrap your ribs and return the the grill.",
      "mode": "Smoke",
      "notify": true,
      "pause": true,
      "timer": 180,
      "trigger_temps": {
        "primary": 0,
        "food": [0, 0]
      }
    },
    {
      "hold_temp": 250,
      "message": "Remove ribs from the wrap, and place on the grill.",
      "mode": "Hold",
      "notify": true,
      "pause": true,
      "timer": 120,
      "trigger_temps": {
        "primary": 0,
        "food": [0, 0]
      }
    },
    {
      "hold_temp": 350,
      "message": "Ribs are finished!",
      "mode": "Hold",
      "notify": true,
      "pause": false,
      "timer": 60,
      "trigger_temps": {
        "primary": 0,
        "food": [0, 0]
      }
    },
    {
      "hold_temp": 0,
      "message": "",
      "mode": "Shutdown",
      "notify": false,
      "pause": false,
      "timer": 0,
      "trigger_temps": {
        "primary": 0,
        "food": [0, 0]
      }
    }
  ]
}
```

__['ingredients']__

Human readable list of ingredients, with optional images. 

+ __name__ - (String) The name of the ingredient to display.
+ __quantity__ - (String) The quantity of that item.
+ __assets__ - (List of Strings) Optional list of strings of filenames, of the images that are to be displayed with this ingredient.  These images and their corresponding thumbnail images must be in the '/assets' and '/assets/thumbnails' folders.

__['instructions']__

Human readable list of instructions, with optional images.

+ __text__ - (String) Text for the instruction step.
+ __ingredients__ - (List of Strings) Optional list of ingredients required for this instruction step.  Should correspond to an 'name' of ingredient in 'ingredients' list. 
+ __assets__ - (List of Strings) Optional list of strings of filenames, of the images that are to be displayed with this instruction step.  These images and their corresponding thumbnail images must be in the '/assets' and '/assets/thumbnails' folders.
+ __step__ - (Integer) Optionally link to a step in the 'steps' data.

__['steps']__

Machine readable list of steps for PiFire to follow for this recipe.  Each step can have triggers, which will optionally fire a notification and optionally move to the next step in the list (unless pause is indicated).  If no triggers and no pause is set, this step will continue until it completes (i.e. Startup & Shutdown) or will remain in that step until stopped/cancelled.  

+ __hold_temp__ - (float/int) Hold Temperature specifically for Hold Mode.
+ __message__ - (String) Notification message to be sent after trigger. 
+ __mode__ - (String) Grill mode (i.e. Startup, Smoke, Hold, Shutdown).
+ __notify__ - (Boolean) If true, send notification on trigger.
+ __pause__ - (Boolean) If true, pause after trigger to wait for user input.
+ __timer__ - (Integer) Timer (minutes) trigger.  If set to 0, timer trigger is disabled.
+ __trigger_temps__ - Temperature triggers.  If set to 0, temperature trigger is disabled for that probe.  If set to a value, the step will trigger when that temperature is achieved.  Multiple triggers can be set and the first target to be achieved, will trigger the step. 
  + __primary__ - (float/int)
  + __food__ - Ordered List of Food Probe Trigger Temps [(float/int)]

### Comments

The comments file is used to store comments and notes about the recipe.  This file may contain an empty list if no comments are added. (i.e. '[]')  

```json
[
  {
    "assets": [
      "6b809ae7-4d01-11ed-b360-b827eb4fc638.jpg",
      "7ca54370-4d01-11ed-bded-b827eb4fc5f9.jpg"
    ],
    "date": "2022-11-15",
    "edited": "2022-11-15 20:25",
    "id": "c00ca782-4d01-11ed-80dc-b827eb4fc620",
    "rating" : 5,
    "text": "This is one of my go-to weekend barbeque recipes. The whole family loves it.",
    "time": "20:22",
    "username": "nebhead"
  },
  {
    "assets": [],
    "date": "2022-11-15",
    "edited": "",
    "id": "24ac3854-4d02-11ed-9420-b827eb4fc613",
    "rating" : 4,
    "text": "1. This recipe is a little different from others, in that it has a higher temp for the wrapping stage.\n2. Don't forget to wrap with the meat side down for best results.\n3. Finish last half an hour at 350F",
    "time": "20:25",
    "username": "nebhead"
  }
]
```

+ __assets:__ List of strings of filenames, of the images that are to be displayed with this comment.  These images and their corresponding thumbnail images must be in the '/assets' and '/assets/thumbnails' folders.

+ __date:__ String of the date that the comment was first entered. Format "YYYY-MM-DD".

+ __edited:__ String of the date that the comment was edited last in the format "YYYY-MM-DD HH:MM".

+ __id:__ (String) Unique ID for the comment.

+ __rating:__ (float) [0-5] Scale for rating the recipe. (0.5 steps)

+ __text:__ Raw string of the comment itself.  The string can include line breaks with the '\n' null character.

+ __time:__ String of the time the comment was first entered.  Format "HH:MM".  

+ __username:__ (string) Username author of the comment.

#### Assets

The assets file stores pointers to all of the asset (image) files in the assets folder.  If there are no assets in this file, the list may be empty (i.e. '[]').  Here is an example of the format.  

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

+ __filename:__ String of the filename for the asset/media that is stored in the '/assets' folder.  [Required]

+ __id:__ String of the unique id assigned to this asset.  [Required]

+ __type:__ String of the filetype for this asset.  [Required]

Others notes about assets/media stored in the file.  

```note
Image files added to the recipe file are rotated, resized (800x600 maximum size) and converted to JPEG format by PiFire automatically.  When images are added to PiFire, they will automatically have a 128x128 thumbnail created, and added to the '/assets/thumbnails/' folder with a corresponding name.
```
