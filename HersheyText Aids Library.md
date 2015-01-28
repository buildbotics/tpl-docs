# HersheyText Aids Library
## Overview
The Hertext Aids Library (HertextTextAids.js) is provided to allow tplang to use the fonts library known as HersheyText.js.  The HersheyText.js fonts library was provided by the [Evil Mad Scientist Laboratories](http://www.evilmadscientist.com/2014/hershey-text-js/).  HersheyText.js is provided with the [Cambotics](http://openscam.org) distribution along with [tplang](http://tplang.org).

At present, HersheyTextAids.js has only one function, which is getLineOfText(TC).  

## Style
HersheyTextAids.js style is described as follows:
* Functions accept and return points in object form.  For instance, a point at x = 1 and y = 2 would be given as {X: 1, Y: 2}.
* Each function accepts a single argument which is an object.  The required properties differ among the various functions.
* Functions return a value of 0 if successful and -1 if not successful.
* Functions returning -1 throw an error string that suggests the problem encountered.
* Functions returning 0 add resulting properties to the argument object.  The resulting property names differ among functions.

## Functions
### getLineOfText(TC)
#### Description
getLineOfText(TC) accepts a line of text and returns a list of paths that can be used to cut the text.  getLineOfText(TC) accepts a single argument (TC).  The properties added to TC define addributes of the resulting paths and are described in the Arguments section.  getLineOfText(TC) can accept any number of characters (including spaces and tabs), but will only process one line at at time. getLineOfText(TC) accepts any of the 23 fonts available in the HersheyText.js library.

####Example
The following code example cuts the "Hello World!" string in the "Script 1-stroke" font.  The character spacing is set to -2 to cause the script characters to move a bit closer to give the appearance that the cursive characters are connected together. The size of the resulting paths are scaled up by a factor of 5, and the size of spaces between words is 5. 

```
var clipper = require('clipper');
var ca = require('ClipperAids');
var hf = require('hersheytext');
var ha = require('HersheyTextAids');
var cutter = require('CuttingAids');

var line = {};
line.text = 				"Hello World!";
line.spacing = 			-2;
line.scale = 				5;
line.font = 				"Script 1-stroke";
line.spaceSize = 		5;
ha.getLineOfText(line);
for (var i = 0; i < line.paths.length; i++)
	cutter.cutPath(line.paths[i],safeHeight,depth);
```

The following image shows the [Cambotics](http://openscam.org) simulation of the resulting [g-code](http:reprap.org/wiki/G-code).
<img src = "https://github.com/buildbotics/tpl-docs/blob/master/images/helloworld.png" height="300" width = "400">

####Arguments
getLineOfText(TC) accepts a single argument TC, which is an object containing the following properties:
* TC.text - TC.text contains the input string of text.  It must be at least one character long.  It can handle any characters with ASCII codes from 0 to 99.  In addition, it can handle spaces and tabs.  It does not handle new line characters.
* TC.font - TC.font is a string that contains one of the twenty-three fonts available in the HersheyText.js library.  The possible fonts include "Astrology", "Script 1-stroke (alt)", "Cyrillic", "Sans 1-stroke","Sans bold", "Gothic English", "Gothic German", "Gothic Italian", "Greek 1-stroke", "Japanese", "Markers", "Math (lower)", "Math (upper)", "Music", "Meteorology", "Script medium", "Script 1-stroke", "Symbolic", "Greek medium", "Serif medium italic", "Serif bold italic", "Serif medium", and "Serif bold".  TC.font is optional and if not provided will default to "Sans 1-stroke".
* TC.scale - TC.scale is a number that specifes the amount by which the size of the resulting cuts will be changed.  For instance, a scale value of 2 will cause the cut paths to twice as big as the original HersheyText fonts in HersheyText.js.  TC.scale is optional and if not provided will default to 1.
* TC.spacing - TC.spacing is a number that specifies the number of units that will be placed between each character in the resulting list of paths.  TC.spacing is scaled by the amount specified in TC.scale.  TC.spacing is optional and if not provided will default to 0.
* TC.spaceSize - TC.spaceSize is a number that specifies the number of units to place in between characters separated by a space.  TC.spaceSize is scaled by the amount in TC.scale.  TC.spaceSize is optional and if not provided will default to 5.
* TC.tabInterval - TC.tabInterval is a number that specifies the interval where tabs will appear. For instance, if TC.tabInterval is set to 100, then a tab characters will cause the text of the next character to be moved ahead to the next position that is a multiple of 100.  TC.tabInterval is scaled by the amount specified in TC.scale.  TC.tabInterval is optional and if not provided will default to 20.

####Results
getLineOfText(TC) returns 0 if no errors are detected and -1 if an error is detected.  The following properties will be set in the argument object (TC) depending on whether errors are detected:
TC.paths - If no error is detected, TC.paths will contain a list of paths.  Each path within the list will be a list of point objects in the form of {X: x Y: y}.  If an error is detected, TC.paths remains unchanged.
TC.width - If no error is detected, TC.width will contain a number that is the overall width of the list of paths that is returned in TC.paths.
TC.height - If no error is detected, TC.height will contain a number that is the overall height of the list of paths that is returned in TC.paths.

####Error Messages
If an error is detected, getLineOfText(TC) returns -1 and throws one of the following values:
* "ARGUMENT\_NOT\_PROVIDED" - Paths could not be created because the required argument object (TC) was not provided.
* "ARGUMENT\_INVALID" - Paths could not be created because the argument that was provided was not an object.
* "TEXT\_NOT\_PROVIDED" - Paths could not be created because the text property (TC.text) was not present in the argument object (TC).
* "TEXT\_INVALID" - Paths could not be created because the text property (TC.text) that was provided in the argument object (TC) was not a string.
* "TEXT\_STRING\_EMPTY" - Paths could not be created because the text string that was provided on the text object (TC.text) was empty.
* "FONT\_INVALID" - Paths could not be created because the value provided in the TC.font object was not a string.
* "FONT\_NOT\_SUPPORTED" - Paths could not be created because the string that was provided in the TC.font property was not one of the following strings, which represent fonts provided by the HersheyText.js library:
  * "Astrology"
  * "Script 1-stroke (alt)"
  * "Cyrillic"
  * "Sans 1-stroke"
  * "Sans bold"
  * "Gothic English"
  * "Gothic German"
  * "Gothic Italian"
  * "Greek 1-stroke"
  * "Japanese"
  * "Markers"
  * "Math (lower)"
  * "Math (upper)"
  * "Music"
  * "Meteorology"
  * "Script medium"
  * "Script 1-stroke"
  * "Symbolic"
  * "Greek medium"
  * "Serif medium italic"
  * "Serif bold italic"
  * "Serif medium"
  * "Serif bold"
* "SCALE\_INVALID" - Paths could not be created because the TC.scale property that was provided was not a number.
* TEXT\_SPACING\_INVALID" - Paths could not be created because the TC.spacing property that was provided was not a number.
* "SPACE\_SIZE\_INVALID" - Paths could not be created because the TC.spaceSize property that was provided was not a number.
* "TAB\_INTERVAL\_INVALID" - Paths could not be created because the TC.tabInterval property that was provided was not a number.


