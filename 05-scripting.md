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

The prototypical "hello world" script for JavaScript is:

```javascript
console.log("Hello, world!");
```

This will print "Hello, world!" to the terminal and if you start Stellarium in a terminal you will, in fact, see this message printed there.
But you probably want to see messages in Stellarium, so instead you can use the `core.debug()` function from the Stellarium APIto print messages to the script console log. For example:

```javascript
core.debug("Hello, Universe!");
```

```{figure} _images/stel-debug.png
:name: console_log
:class: dark-light
:width: 100%
:align: center
Stellarium Console Log
```

As the name implies, the `core.debug()` function is meant for debugging purposes and can be useful for tracking down errors in your script.

```{note}
All statements in JavaScript files must end with a semicolon. This is a common source of errors in scripts, so be sure to include semicolons at the end of each statement.
```

### Common JavaScript Data Structures

You can define variables and constants in your script to store values that you want to use later. For example:

```javascript
const BrightRed = core.vec3f(0.8314, 0.1647, 0.1647);  // #D42A2A

var date = "2026-05-01T00:00:00"; // Date and time in ISO 8601 format
```

```{note}
There are three (or four) ways to define variables in JavaScript: `var`, `let`, and `const`. Each has subtle differences mostly related to scope and mutability. People have strong opinions on this. You can read about it [here](https://www.freecodecamp.org/news/var-let-and-const-whats-the-difference/) if you are interested.
```

You can also define arrays and objects to store more complex data structures. For example:

```javascript
// An array of the names of the planets. (Pluto is a minor planet, sorry Pluto.)
const planets = ["Mercury", "Venus", "Earth", "Mars", "Jupiter", "Saturn", "Uranus", "Neptune"];
```

You can access the elements of an array using their index, for example: `planets[0]` returns "Mercury" (JavaScript uses zero-based indexing). You can add elements to an array using the `push` method, for example: `planets.push("Pluto")` would add "Pluto" to the end of the array. You can remove the last element of an array using `planets.pop()`, or the first using `planets.shift()`.

To delete a specific element from an array, you can use the `splice` method, for example:

```javascript
// get the index of "Pluto" and delete one element beginning at that index
index = planets.indexOf("Pluto");
if (index !== -1) {
    planets.splice(index, 1);
}
```

An array can be iterated over using a loop, for example:

```javascript
for (var i = 0; i < planets.length; i++) {
    core.debug("Hello, " + planets[i] + "!");
}
```

This is the basic structure of a loop. The first part `var i = 0` initializes the loop variable `i` (for index) to 0. The second part `i < planets.length` is the conditional and the loop will continue as long as this condition is true. The third part `i++` increments the loop variable `i` by 1 after each iteration of the loop. For each iteration of the loop, it will concatenate the string "Hello, " with the name of each planet and "!" and print a greeting to the console log.

```{note}
Blocks of code in JavaScript are delimited with curly braces `{}`. This is also a common source of errors in scripts, so be sure to include the opening and closing curly braces for all blocks of code, such as function definitions and loops.
```

Another common data structure is an object, which is a collection of key-value pairs. For example:

```javascript
// an object with the parameters for displaying an image
const alf_CMa = {
    id: "alf_CMa",
    path: "../images/alf_CMa.png",
    ra: "6h 45m 08.0s",
    dec: "-16d 43m 30.0s",
    angSize: 1.5, // arcminutes
    rotation: 0,  // degrees
    minRes: 2.5,
    maxBright: 14,
    visible: false
};
```

You can access the properties of an object using dot notation, for example: `alf_CMa.ra` returns the string "6h 45m 08.0s".

```{note}
You will see some examples of common JavaScript constructs throughout this guide, but I won't go into detail about them. There are many free resources available online to learn the basics, such as [this one](https://www.geeksforgeeks.org/javascript/javascript-tutorial/), or [this one](https://www.freecodecamp.org/news/learn-javascript-for-beginners/). If you get stuck, ChatGPT knows JavaScript an even a little about Stellarium.
```

### Functions

Functions are reusable blocks of code that perform a specific task. You can define your own functions in your script to organize your code and make it more modular. For example:

```javascript
// a function to pause the script until the user presses the 'L' key
function pauseKey(label = true) {
    var timerate = core.getTimeRate();
    core.setTimeRate(0.05);
    L = LabelMgr.labelScreen("Paused ('L' to continue)", 20, core.getScreenHeight() - 60,
        visible = false, fontSize = 18, fontColor = "#666666");
    if (label) {
        LabelMgr.setLabelShow(L, true);
    }
    do {
        core.wait(1);
    }
    while (core.getTimeRate() < 0.1);
    core.setTimeRate(timerate);
    LabelMgr.deleteLabel(L);
}
```

You would call this function with `pauseKey()`. It will pause the script until the user presses the 'L' key. It does this by saving the current time rate, setting the time rate to a slow value to effectively pause the script, and then waiting in a loop until the time rate is increased again (by pressing 'L'). After resuming, it restores the original time rate. It has an optional argument `label` which defaults to `true`.

### Include statements

You can include other scripts in your script using the `include` function. Included scripts contain variables and functions that can be used elsewhere in your script. This is useful for organizing your code and reusing common functions across multiple scripts. Included scripts typically have a `.inc` extension to indicate that they are meant to be included in other scripts rather than run on their own. These will not be listed in the scripts tab and can not be run directly.

The `include` function takes the path to the script you want to include as an argument.

```javascript
include("common_objects.inc");
```

An example of what an include file might contain.

```javascript
// Name: common objects include file
// License: Public Domain
// Author: Matthew Gates, Alexander Wolf
// Description: Define arrays containing object name (in English).  
//              Defines: planets, constellations, nebulae

// Constellations
const constellations = ConstellationMgr.getConstellationsEnglishNames();

const circumpolar = ["UMi", "UMa", "Dra", "Cep", "Cas", "Cam"];

const zodiac = ["Ari", "Tau", "Gem", "Can", "Leo",
    "Vir", "Lib", "Sco", "Sgr", "Cap", "Aqr", "Psc"];

// Messier and Caldwell lists
const messier = [];
const caldwell = [];

for (let i = 1; i <= 110; i++) {
    messier.push("M" + i);

    if (i <= 109) {
        caldwell.push("C" + i);
    }
}
```

The first line of this example the script calls the `getConstellationsEnglishNames` function of the `ConstellationMgr` object to get an array of the names of all the constellations in English and assigns it to the variable `constellations`.

The next two lines define arrays for the circumpolar and zodiac constellations.

The last part defines two empty arrays for the Messier and Caldwell objects and uses a loop to generate the names of the objects.


### Main script

The main part of the script consists primarily of a series of calls to the Stellarium API to control various aspects of Stellarium. This is where you would put the main logic of your script, such as setting the location, time, and landscape, and then performing actions like pointing to objects, changing the field of view, and so on.

The API modules are automatically loaded by the Stellarium scripting engine, so you can use any of the available functions and properties in your script without needing to import or require anything. You can find a list of all the available API modules and their functions and properties in the Stellarium scripting documentation.

The [`core`](https://stellarium.org/doc/26.0/classStelMainScriptAPI.html) module contains functions and properties related to the core functionality of Stellarium. They can be accessed using the `core` object, for example:

```javascript
 core.setDate("2026-05-01T00:00:00", spec = "local");
 ```

The `core` functions are more general and usually easier to use so I recommend using those when possible. Other modules have functions useful for more specific tasks.

Take a look at the following modules that are available for use in scripts. Some you will use a lot, while others you may never use.

- [`AsterismMgr`](https://stellarium.org/doc/26.0/classAsterismMgr.html)
- [`ConstellationMgr`](https://stellarium.org/doc/26.0/classConstellationMgr.html)
- [`CustomObjectMgr`](https://stellarium.org/doc/26.0/classCustomObjectMgr.html)
- [`HighlightMgr`](https://stellarium.org/doc/26.0/classHighlightMgr.html)
- [`GridLinesMgr`](https://stellarium.org/doc/26.0/classGridLinesMgr.html)
- [`LabelMgr`](https://stellarium.org/doc/26.0/classLabelMgr.html)
- [`MarkerMgr`](https://stellarium.org/doc/26.0/classMarkerMgr.html)
- [`LandscapeMgr`](https://stellarium.org/doc/26.0/classLandscapeMgr.html)
- [`SporadicMeteorMgr`](https://stellarium.org/doc/26.0/classSporadicMeteorMgr.html)
- [`NebulaMgr`](https://stellarium.org/doc/26.0/classNebulaMgr.html)
- [`ScreenImageMgr`](https://stellarium.org/doc/26.0/classScreenImageMgr.html)
- [`SolarSystem`](https://stellarium.org/doc/26.0/classSolarSystem.html)
- [`StarMgr`](https://stellarium.org/doc/26.0/classStarMgr.html)
- [`StelActionMgr`](https://stellarium.org/doc/26.0/classStelActionMgr.html)
- [`StelAudioMgr`](https://stellarium.org/doc/26.0/classStelAudioMgr.html)
- [`StelVideoMgr`](https://stellarium.org/doc/26.0/classStelVideoMgr.html)
- [`StelMovementMgr`](https://stellarium.org/doc/26.0/classStelMovementMgr.html)
- [`StelSkyDrawer`](https://stellarium.org/doc/26.0/classStelSkyDrawer.html)
- [`StelSkyLayerMgr`](https://stellarium.org/doc/26.0/classStelSkyLayerMgr.html)
- [`SpecialMarkersMgr`](https://stellarium.org/doc/26.0/classSpecialMarkersMgr.html)
- [`MilkyWay`](https://stellarium.org/doc/26.0/classMilkyWay.html)
- [`ZodiacalLight`](https://stellarium.org/doc/26.0/classZodiacalLight.html)
- [`HipsMgr`](https://stellarium.org/doc/26.0/classHipsMgr.html)
- [`NomenclatureMgr`](https://stellarium.org/doc/26.0/classNomenclatureMgr.html)

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

Location can be set either by name or by coordinates. If set by name the location must be defined in the location database.

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

#### Selecting Objects and Navigating the Sky

The `StelMovementMgr` module has a few functions to let you quickly set the view. You can use `lookNorth()`, `lookSouth()`, `lookEast()`, `lookWest()`,`lookTowardsNCP()`, and `lookZenith()` to quickly set the view to the cardinal directions, the NCP or the zenith. The `zoomTo(fov, duration)` function can be used to set the field of view to a specific value in degrees over a specified duration in seconds. To reset the view to the default view, you can use `core.goHome()`.

```javascript
// This example demonstrates use of the StelMovementMgr module to set the view.

core.goHome(); // reset the view to the default
core.wait(2); // wait for 2 seconds

StelMovementMgr.lookEast();
core.wait(2);

StelMovementMgr.lookSouth();
core.wait(2);

StelMovementMgr.lookWest();
core.wait(2);

StelMovementMgr.lookNorth();
core.wait(2);

StelMovementMgr.lookTowardsNCP();
GridLinesMgr.setFlagEquatorGrid(true); // turn on the equatorial grid to show the NCP
StelMovementMgr.zoomTo(30, 4); // zoom to 30 degrees over 4 seconds
core.wait(6); // wait for 6 seconds, 4 seconds for the zoom to complete and 2 seconds more.
GridLinesMgr.setFlagEquatorGrid(false);

StelMovementMgr.lookZenith();
GridLinesMgr.setFlagAzimuthalGrid(true); // turn on the azimuthal grid to show the zenith
StelMovementMgr.zoomTo(180, 4);
core.wait(4);
GridLinesMgr.setFlagAzimuthalGrid(false);
```

```{note}
If you invoke `core.goHome()` from the zenith, it will mess up the horizon line (unless the home position is the zenith, in which case it's fine). To fix this, you can move slightly off the zenith before calling `core.goHome()`.
```

##### Navigating by Coordinates

You can use `core.moveToAltAzi(alt, az, duration)` and `core.moveToRaDec(ra, dec, duration)` to set the view to a specific azimuth and altitude or RA and Dec. The RA and Dec here are apparent. There are functions for J2000 and galactic coordinates as well. You can set the view or get the current view in any of these coordinate systems. You can set the view to either "azimuthal" or "equatorial" using the `core.setMountMode("mode")` function. Azimuthal will give you a view relative to your local horizon, while the default equatorial view is North up, East left.

##### Moving to Objects Specified by Name

Another way to navigate is to select or move to specific objects. The function `core.moveToObject(object, duration)` will move the view to the specified object over the specified duration in seconds. The object can be specified by name or designation. They can be stars, solar system objects, deep sky objects, or constellations.

##### Selecting Objects and Zooming

You can select objects by name or designation using the `core.selectObjectByName(object)` function. You can then go to the object using `core.moveToSelectedObject(duration)`. You can use the `StelMovementMgr.autoZoomIn()` and `StelMovementMgr.autoZoomOut()` functions to zoom in and out on the selected object. Selected objects will remain in the center of the screen as the time changes, so if you fast forward the time and want the horizon to remain fixed, use `StelMovementMgr.deselection()` to deselect all objects first.

```javascript
// This example demonstrates how to select objects by name and show them.

include("common_objects.inc");

core.setDate("2026-06-01T21:00:00", spec = "local");

// set the view to the north twenty degrees above the horizon
core.moveToAltAzi(20, 0, 4);
StelMovementMgr.zoomTo(60, 4);
core.wait(4);

// fast forward 30 minutes
core.setTimeRate(800);
core.waitFor("+30minutes");
core.setTimeRate(0);
core.wait(4);

// select Polaris and move to it
core.selectObjectByName("Polaris");
core.moveToSelectedObject(); // default duration is 1 second
StelMovementMgr.autoZoomIn();
core.wait(4);
StelMovementMgr.autoZoomOut();

// set up constellation lines and equatorial grid
ConstellationMgr.setFlagIsolateSelected(true);
ConstellationMgr.setFlagConstellationPick(false);
ConstellationMgr.deselectConstellations();
ConstellationMgr.setFlagLines(true);

StelMovementMgr.lookTowardsNCP();
GridLinesMgr.setFlagEquatorGrid(true);
StelMovementMgr.zoomTo(80, 4);

// loop through the circumpolar constellations while fast forwarding
core.setTimeRate(800);

// circumpolar is an array defined in the common_objects.inc 
for (var i = 0; i < circumpolar.length; i++) {
    core.selectConstellationByName(circumpolar[i]);
    StelMovementMgr.deselection();
    core.wait(2);
}

core.waitFor("+3hours", spec = "local");
core.setTimeRate(0);
```

#### Lines, Constellations, Labels, and Markers

  TODO:

#### Images and Videos

  TODO:

## A Few General Tips

If you are new to JavaScript or working with Stellarium scripts, the most important thing to remember is if it works it's good enough. Don't worry about writing perfect code.

But... A few things to keep in mind:

- Document your code. It will make your life easier and help others understand it.
- Don't repeat yourself. Use functions to organize your code and make it reusable.
- RTFM. The Stellarium scripting documentation is not so good, but it's there if you need it.
- Steal other people's stuff.
- Ask for help. Help if asked.

That is all.
