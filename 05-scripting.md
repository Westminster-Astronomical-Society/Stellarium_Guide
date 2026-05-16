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
    "Virgo", "Libra", "Scorpius", "Sgt", "Cap", "Aqr", "Psc");

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

