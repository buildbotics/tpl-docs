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
CLIP.subject = [r.rect], CLIP.clip = m.newPaths, CLIP.type = "UNION";
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

##Functions
###clip(CLIP)
####Description
clip(CLIP) allows users to use the clipping capabilities provided in the [Clipper.js](http://sourceforge.net/projects/jsclipper/) javascript library.  These clipping functions allow creating unions, intersections, differences, and exclusive or's between two lists of paths.  clip(CLIP) accepts a single argument, which is an object that contains the list of paths that will be the subject of clipping, a second list of paths that will be used to clip the subject paths, and the type of clipping that will occur.

####Example
The following code approximates a circle by creating a 50 sided polyhedron and a star with rounded outer vertexes.  It then performs each of the four clipping operatoins using the star as the subject and the circle as the clip.
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

var p = {},s = {};
p.radius = 100, p.count = 50;
da.polyhedron(p);
s.outerRadius = 150,s.innerRadius = 50,s.numberOfPoints = 5;
da.makeStar(s);
var c = {};
c.subject = [s.star], c.clip = [p.polyhedron];
c.type = "INTERSECTION";
ca.clip(c);
for (var i = 0; i < c.solutions.length; i++) cutter.cutPath(c.solutions[i],safeHeight,depth);

translate(350,0,0);
c.type = "UNION";
ca.clip(c);
for (var i = 0; i < c.solutions.length; i++) cutter.cutPath(c.solutions[i],safeHeight,depth);

translate(350,0,0);
c.type = "DIFFERENCE";
ca.clip(c);
for (var i = 0; i < c.solutions.length; i++) cutter.cutPath(c.solutions[i],safeHeight,depth);

translate(350,0,0);
c.type = "XOR";
ca.clip(c);
for (var i = 0; i < c.solutions.length; i++) cutter.cutPath(c.solutions[i],safeHeight,depth);
```

The resulting cuts, which were simulated in the [Camotics](http://openscam.org) simulator are shown here.

<img src = "https://github.com/buildbotics/tpl-docs/blob/master/images/allcliptypes.png" height="300" width = "400">

####Arguments
clip(CLIP accepts a single argument, CLIP.  CLIP is an object with properties that include the list of paths to be clipped (CLIP.subject), the list of paths to clip with (CLIP.clip) and the type of clipping to perform (CLIP.type).  CLIP has the following properties:
* CLIP.subject - CLIP.subject is a list of paths.  Each path within the list is a list of points of the form {X: x, Y: y}. CLIP.subject contains the subject of the clipping operation.  In the code example above, the subject was the star because we wanted to subtract the circle from the star.  CLIP.subject is required and must be a defined object, but it is allowed to be empty.  In other words, is allowed to be a list containing a single empty list like [[]].  This is because the union operation (clipper.ClipType.ctUnion) does not distinguish between the subject and clip and simply adds everything together in both the subject and the clip, and it is often easier to put all paths into either the subject or the clip for this operation.
* CLIP.clip - CLIP.clip is a list of paths.  Each path within the list is a list of points of the form {X; x, Y; y}.  CLIP.clip contains the paths that will be used to clip the subject.  In the code example above, the circle was the clip because we wanted to subtract the circle from the star and leave the remaining parts of the star that were outside the circle intact.  CLIP.clip is required and must be a defined object, but it is allowed to be empty.  In other words, is allowed to be a list containing a single empty list like [[]].  This is because the union operation (clipper.ClipType.ctUnion) does not distinguish between the subject and clip and simply adds everything together in both the subject and the clip, and it is often easier to put all paths into either the subject or the clip for this operation.
* CLIP.type - CLIP.type is string that specifies the type of clipping operation to perform.  It can be one of the following strings:
  * "INTERSECTION" - If CLIP.type is equal to "INTERSECTION" the resulting paths will form the intersection of the subject and the clip. In the example above the leftmost cut is the intersection of the star with the circle.  Notice that only the area that is inside both polygons is left to from the star with the points cut off.
  * "UNION" - If CLIP.type is equal to "UNION", the resulting paths will include any area that was any of the polygons.  In the example above, the union of the star and circle was formed and the result is shown in the second cut.
  * "DIFFERENCE" - If CLIP.type is equal to "DIFFERENCE", the resulting paths will be the areas in the subject polygons minus the areas in the clip polygons.  In the example, the third image is the difference and shows only the points of the star because the circle was subtracted from the star.
  * "XOR" - If CLIP.type is equal to "XOR", only the areas that fall in either the subject or the clip will remain in the result.  This is shown in the fourth image in the example.  The fourth image looks like a circle and a star, but it is actually ten polygons that make up the opposite of the first image.
####Results
clip(CLIP) will return 0 if no errors are detected and -1 if an error is detected.  The following properties are loaded into the argument object, CLIP depending on whether an error is detected.
* CLIP.solutions - If no error is detected, CLIP.solutions will be loaded with the list of paths that result from the clipping operation.  CLIP.solutions is a list of paths.  Each of the paths within the list of paths is a list of point objects in the form of {X: x, Y: y}.

####Error Messages
If an error is detected, clip(CLIP) returns -1 and throws one of the following error messages:
* "ARGUMENT\_NOT\_PROVIDED" - The clip operation could not run because no argument (CLIP) was provided.
* "INVALID\_ARGUMENT" - The clip operationn could not run because the arugment that was provided was not an object.
* "SUBJECT\_NOT\_PROVIDED" - The clip operation could not run because the CLIP.subject was not provided.  At a minimum, the subject must be an empty list of lists.
* "INVALID\_SUBJECT" - The clip operation could not run because the CLIP.subject that was provided was not an object"
* "CLIP\_NOT\_PROVIDED" - The clip operation could not run because the CLIP.clip was not provided.  At a minimum, the subject must be and empty list of lists.
* "INVALID\_CLIP" - The clip operation could not run because the CLIP.clip property that was provided was not an object.
* CLIP\_TYPE\_NOT\_PROVIDED" - The clip operation could not run because the CLIP.type property was not provided.
* "INVALID\_CLIP\_TYPE" - The clip operation could not run because the CLIP.type property that was provided was not one of the following strings:
  * "INTERSECTIN"
  * "UNION"
  * "DIFFERENCE"
  * "XOR"





