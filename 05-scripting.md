# Writing Stellarium Scripts

There is a short introduction to writing scripts in the Stellarium documentation and an API reference at the website that lists all the available functions and properties. The best way to learn how to write scripts is to look at the example scripts to see how they work and modify them to suit your needs.

## Basic script structure

A Stellarium script is a JavaScript file with a `.ssc` extension. The script can contain any valid JavaScript code, but it typically consists of a series of commands that control various aspects of Stellarium.

### Script header

The script header is a comment block at the beginning of the script that provides metadata about the script. The header is optional, but this is the information that is displayed in the script information section when you select the script in the scripts tab.

```javascript
// Title: My Script
// Author: Your Name
// License: MIT (or whatever) 
// Version: 1.0
// Description: This is a description of my script.
```

```{note}
Comments in scripts are denoted by `//` and are ignored by the JavaScript interpreter.
```

### Include statements

You can include other scripts in your script using the `include` function. Included scripts contain variables and functions that can be used elsewhere in your script. This is useful for organizing your code and reusing common functions across multiple scripts. Included scripts typically have a `.inc` extension to indicate that they are meant to be included in other scripts rather than run on their own. These will not be listed in the scripts tab and can not be run directly.

The `include` function takes the path to the script you want to include as an argument.

```javascript
include("common_objects.inc");
```

```{note}
All statements in JavaScript files must end with a semicolon. This is a common source of errors in scripts, so be sure to include semicolons at the end of each statement.
```

An example of what an include file might contain.

```javascript
// Name: common objects include file
// License: Public Domain
// Author: Matthew Gates, Alexander Wolf
// Description: Define arrays containing object name (in English).  
//              Defines: planets, constellations, nebulae

// Constellations
var constellations = ConstellationMgr.getConstellationsEnglishNames();

var circumpolar = new Array("UMi", "UMa", "Dra", "Cep", "Cas", "Cam");

var zodiac = new Array("Ari", "Tau", "Gem", "Can", "Leo",
    "Vir", "Lib", "Sco", "Sgr", "Cap", "Aqr", "Psc");

// Messier and Caldwell lists
var messier = new Array();
var caldwell = new Array();
for (var i = 1; i <= 109; i++) {
    messier.push("M" + i);
    caldwell.push("C" + i);
}
messier.push("M110");
```

The first line of this example the script calls the `getConstellationsEnglishNames` function of the `ConstellationMgr` object to get an array of the names of all the constellations in English and assigns it to the variable `constellations`.

The next two lines define arrays for the circumpolar and zodiac constellations.

The last part defines two empty arrays for the Messier and Caldwell objects and uses a loop to generate the names of the objects.

The arrays are lists of objects that can be accessed by index or iterated over in a loop. For example, you could use the `zodiac` array to point to select and display each constellation in turn.

```{note}
There are three (or four) ways to define variables in JavaScript: `var`, `let`, and `const`. Each has subtle differences mostly related to scope and mutability. People have strong opinions on this. You can read about it [here](https://www.freecodecamp.org/news/var-let-and-const-whats-the-difference/) if you are interested.
```

### Variables and functions

While it is not strictly required to define variables and functions before using them in a script, sometimes it is, and it is generally good practice to do so in all cases.

```javascript
include("common_objects.inc");

// Define some colors
const White = core.vec3f(1.0, 1.0, 1.0);                  // #FFFFFF
const Gray = core.vec3f(0.4, 0.4, 0.4);                   // #666666
const Black = core.vec3f(0.0, 0.0, 0.0);                  // #000000

const BrightRed = core.vec3f(0.8314, 0.1647, 0.1647);     // #D42A2A
const TrueGreen = core.vec3f(0.00000, 0.56863, 0.00000);  // #009100
const Cobalt = core.vec3f(0.00000, 0.28627, 0.56863);     // #004991


const dtg = "2026-05-01T00:00:00"; // Date and time in ISO 8601 format

const epochTime = Date.now(); // Current date and time in milliseconds since the Unix epoch
const julianDate = getJulianDate(epochTime); // Current date and time in Julian date format


function pauseKey(label = true) {
    var timerate = core.getTimeRate();
    core.setTimeRate(0.05);
    L = LabelMgr.labelScreen("Paused ('L' to continue)", 20, core.getScreenHeight() - 60,
        visible = false, fontSize = 18, fontColor = "#666666");
    if (label) {
        LabelMgr.setLabelShow(L, true);
    }
    i = 0;
    do {
        core.wait(1);
        i++;
    }
    while (core.getTimeRate() < 0.1);
    core.setTimeRate(timerate);
    LabelMgr.deleteLabel(L);
}

```

In this example, we define some color variables using the `core.vec3f` function so that they can be referred to by name rather than by their RGB values. We also define a constant variable `dtg` that contains a date and time in ISO 8601 format, and we use the `Date.now()` function to get the current date and time in milliseconds since the Unix epoch, which we then convert to Julian date format using the `getJulianDate` function. These are defined as constants because they are not meant to be changed.

Finally, we define a function called `pauseKey` that pauses the script until the user presses the 'L' key. The function takes an optional argument `label` that determines whether to display a label on the screen while paused. The function saves the current time rate, sets the time rate to a slow value to effectively pause the script, and then waits in a loop until the time rate is increased again (by pressing 'L'). After resuming, it restores the original time rate and deletes the label if it was displayed.

```{note}
Blocks of code in JavaScript are denoted by curly braces `{}`. This is also a common source of errors in scripts, so be sure to include the opening and closing curly braces for all blocks of code, such as function definitions and loops.
```

### Main script

The main part of the script consists primarily of a series of calls to the Stellarium API to control various aspects of Stellarium. This is where you would put the main logic of your script, such as setting the location, time, and landscape, and then performing actions like pointing to objects, changing the field of view, and so on.

The API modules are automatically loaded by the Stellarium scripting engine, so you can use any of the available functions and properties in your script without needing to import or require anything. You can find a list of all the available API modules and their functions and properties in the Stellarium scripting documentation.

The `core` module contains functions and properties related to the core functionality of Stellarium. They can be accessed using the `core` object, for example:

```javascript
 core.setDate(dtg, spec = "local");
 ```

In addition, the following modules are available for use in scripts:

- `AsterismMgr`
- `ConstellationMgr`
- `CustomObjectMgr`
- `HighlightMgr`
- `GridLinesMgr`
- `LabelMgr`
- `MarkerMgr`
- `LandscapeMgr`
- `SporadicMeteorMgr`
- `NebulaMgr`
- `ScreenImageMgr`
- `SolarSystem`
- `StarMgr`
- `StelActionMgr`
- `StelAudioMgr`
- `StelVideoMgr`
- `StelMovementMgr`
- `StelSkyDrawer`
- `StelSkyLayerMgr`
- `SpecialMarkersMgr`
- `MilkyWay`
- `ZodiacalLight`
- `HipsMgr`
- `NomenclatureMgr`

Some, but not all of the plugins are also accessible from scripts.

#### Setting up the Presentation

At the beginning of your script you will typically want to set up the presentation by setting the location, landscape, display options, and the initial view. You can put this in the main part of the script or as a separate function that can be reused in other scripts. For example:

```javascript
// Name: settings.inc
// License: Public Domain
// Author: Christopher Bennett
// Version: 1.0, 2026-05-15
// Description: This file contains settings and utility functions for Stellarium scripts

include("save_state.inc");

function setup() {
    core.debug("Setting up environment");

    // reset view to home and save state.
    core.goHome();
    saveState("RestoreState");

    // az mount, atmosphere, landscape, no lines, labels or markers
    core.clear("natural");

    // clear any existing markers, labels, and screen images
    MarkerMgr.deleteAllMarkers();
    LabelMgr.deleteAllLabels();
    ScreenImageMgr.deleteAllImages();

    // hide the sky layer and remove any existing sky images
    StelSkyLayerMgr.setFlagShow(false);
    var keys = StelSkyLayerMgr.getAllKeys();

    if (keys.length > 2) {
        for (var i = 2; i < keys.length; i++) {
            core.removeSkyImage(keys[i]);
        };
    };

    core.wait(2);
    // core.setObserverLocation("Bear Branch", "Earth");
    core.setObserverLocation(-76.986769, 39.647307, 210, 2, "BBNC", "Earth");
    LandscapeMgr.setCurrentLandscapeName("Bear Branch");

    // Set any other options you want here, for example, grid lines, fonts, colors, etc.

    core.setGuiVisible(false);
}
```

This example will give you a clean environment to work with for your presentation. There is a `save_state.inc` file that comes with Stellarium that contains a function for saving the state of Stellarium so that you can restore it at the end of your presentation. **It does not work with the latest version of Stellarium**, but there is an updated version in the Stellarium_Template repository on the WASI GitHub. To use it call `saveState("RestoreState")` at the beginning of your script and then you can call `restoreState("RestoreState")` at the end of your script to restore the state of Stellarium to what it was before you ran the script.

The `core.clear("natural")` function call will set the view to an azimuthal mount, turn on the atmosphere, and landscape and lines, labels and marker off. other options for this function include:

- starchart : equatorial mount, constellation lines on, no landscape, no atmosphere etc.  and labels & markers on.
- deepspace : like starchart, but no planets, no eq.grid, no markers, no lines.
- galactic : like deepspace, but in galactic coordinate system.
- supergalactic : like deepspace, but in supergalactic coordinate system. 

There is no deleteAll function for SkyImages so they have to be deleted individually. The loop unloads them one by one leaving the first two keys (custom textures and nebulae) loaded.

You can include this file in your main script and call the `setup` function at the beginning of your script to set up the presentation environment.

```javascript
// Title: My Script
// Author: Your Name
// License: MIT (or whatever) 
// Version: 1.0
// Description: This is a description of my script.

include("common_objects.inc");
include("settings.inc");

setup();
```

#### Date and Time

You can set the date and time in your script using the `core.setDate` function. The date and time should be in ISO 8601 format, for example:

```javascript
core.setDate("2026-05-01T00:00:00", spec = "local");
```

Dates can be specified in UTC, local time, or sidereal time by changing the `spec` argument to "utc" or "local". UTC is the default. You can use `setDate("now")` to set the date and time to the current date and time. Relative dates and times can also be specified using the `+` and `-` operators, for example:

```javascript
core.setDate("-1hour"); // subtract one hour from the date and time
core.setDate("now +1day sidereal"); // add one sidereal day to the current date
```

Units for relative dates are second, minute, hour, day, sol, week, month, and year. The sol unit is one day on the planet that is currently selected as the observer's location. The units are in solar time unless the "sidereal" specifier is included.

Dates can also be set using the Julian day number with `core.setJDay(2461177.5)`.

You can set the time rate using the `core.setTimeRate` function. The time rate is a multiplier for the passage of time in Stellarium. For example, a time rate of 1 means that time passes at the normal rate, while a time rate of 60 means that time passes 60 times faster than normal. You can use this to speed up or slow down the passage of time in your presentation. Used in combination with `core.waitFor()`, you can fast forward to a specific date and time in your presentation. For example:

```javascript
core.setDate("2026-05-01T21:00:00", spec = "local");

core.setTimeRate(600); // set time rate to 600x normal speed
core.waitFor("2026-05-02T00:00:00", spec = "local"); // wait for 3 hours to pass
core.setTimeRate(1); // set time rate back to normal
```

Make sure the waitFor date is in the future relative to the current date and time, otherwise the script will wait forever. You can also use `core.waitFor("+3hours")` to wait for a relative amount of time to pass to avoid this problem.

There are quite a few functions for getting, setting, and working with dates and times in the `core` module, so be sure to check the documentation for more information on how to use them in your scripts.

#### Selecting and Viewing Objects

