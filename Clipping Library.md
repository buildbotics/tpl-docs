#Clipping Library
##Overview
The Clipping Library provides a set of functions that allow [tplang](http://tplang.org) programmers to easily take advantage of the powerful [Clipper.js](http://sourceforge.net/projects/jsclipper/) javascript library.  While it is possible to use [Clipper.js](http://sourceforge.net/projects/jsclipper/) directly, it is the intent of this Clipping Library to wrap the most commonly used parts of [Clipper.js](http://sourceforge.net/projects/jsclipper/) into a set of simple functions.  Even still, users are likely to find the [Clipper.js documentation](http://sourceforge.net/p/jsclipper/wiki/documentation/) helpful in gaining a better understanding of this Clipping Library.

##Example
The following code shows how the Clipping Library can be used to get to merge two rectangles that are offet from one another into a single continuous shape and then cut the result.
```
units(METRIC); // units are in inches
feed(30); // feed rate us 30 inches per minute
speed(4000); // spindle speed is 4000 rpm
var bitWidth = 3.125;
var safeHeight = 3;
var depth = 6.4;
tool(1);

var clipper = require('clipper');
var ca = require('ClipperAids');
var da = require('DrawingAids');
var cutter = require('CuttingAids');

var r = {};
r.width = 150, r.height = 100, r.cornerRadius = 10, r.cornerIncrements = 10;
da.makeRectangle(r);
var m = {}
m.paths = [r.rect],m.x=80,m.y=80;
da.moveBy(m);
var CLIP = {};
CLIP.subject = [r.rect], CLIP.clip = m.newPaths;
CLIP.type = clipper.ClipType.ctUnion;
ca.clip(CLIP)
cutter.cutPath(CLIP.solutions[0],safeHeight,depth);
```

The resulting cuts, which were simulated in the [Camotics](http://openscam.org) simulator are shown here.

<img src = "https://github.com/buildbotics/tpl-docs/blob/master/images/rectangleunion.png" height="300" width = "400">

## Style
The Clipper Library functions all use a similar style, which is described as follows:
* Functions accept and return points in object form.  For instance a point at x = 1 and y = 2 would be given as {X: 1, Y: 2}.
* Each function accepts a single argument which is an object.  The required properties differ among the various functions.
* Functions return a value of 0 if successful and -1 if not successful.
* Functions returning -1 throw an error string with suggests the problem encountered.
* Functions returning 0 add a property that includes the result to the argument object.  The resulting property name differs among functions.
