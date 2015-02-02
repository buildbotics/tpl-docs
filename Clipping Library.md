#Clipping Library
## Table of Contents <a name = 'Table of Contents' />

| Section              | SubSection                                              |
|----------------------|:-------------------------------------------------------:|
|[Overview](#Overview) |                                                         |
|                      |[Example](#OverviewExample)                              |
|                      |[Style](#OverviewStyle)                                  |
|[Functions](#Functions)|                                                        |
|                      |[clip(CLIP)](#clip)                                      |
|                      |[offset(OFF)](#offset)                                   |
|                      |[makePathsPolygons(P)](#makePathsPolygons)               |

##Overview <a name = 'Overview' />
The Clipping Library provides a set of functions that allow [tplang](http://tplang.org) programmers to easily take advantage of the powerful [Clipper.js](http://sourceforge.net/projects/jsclipper/) javascript library.  While it is possible to use [Clipper.js](http://sourceforge.net/projects/jsclipper/) directly, it is the intent of this Clipping Library to wrap the most commonly used parts of [Clipper.js](http://sourceforge.net/projects/jsclipper/) into a set of simple functions.  Even still, users are likely to find the [Clipper.js documentation](http://sourceforge.net/p/jsclipper/wiki/documentation/) helpful in gaining a better understanding of this Clipping Library.

[Back to Table of Contents](#Table of Contents)

##Example <a name = 'OverviewExample' />
The following code shows how the Clipping Library can be used to merge two rectangles that are offet from one another into a single continuous shape and then cut the result.
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

[Back to Table of Contents](#Table of Contents)
## Style <a name = 'OverviewStyle' />
The Clipper Library functions all use a similar style, which is described as follows:
* Functions accept and return points in object form.  For instance a point at x = 1 and y = 2 would be given as {X: 1, Y: 2}.
* Each function accepts a single argument which is an object.  The required properties differ among the various functions.
* Functions return a value of 0 if successful.
* If an error is detected functions throw an error message that suggests the problem that was encountered.  That message along with trace can be caught in the try-catch statement.  The error and the trace information can be accessed in the stack property of the err object (e.g. console.log(err.stack)).
* Functions returning 0 may add properties that includes the result to the argument object.  The resulting property names differ among functions.

[Back to Table of Contents](#Table of Contents)
##Functions <a name = 'Functions' />
###clip(CLIP) <a name = 'clip' />
####Description
clip(CLIP) allows users to use the clipping capabilities provided in the [Clipper.js](http://sourceforge.net/projects/jsclipper/) javascript library.  These clipping functions allow creating unions, intersections, differences, and exclusive or's between two lists of paths.  clip(CLIP) accepts a single argument, which is an object that contains the list of paths that will be the subject of clipping, a second list of paths that will be used to clip the subject paths, and the type of clipping that will occur.

[Back to Table of Contents](#Table of Contents)
####Example
The following code approximates a circle by creating a 50 sided polyhedron and a star with five points.  It then performs each of the four clipping operations using the star as the subject and the circle as the clip.
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
clip(CLIP) accepts a single argument, CLIP.  CLIP is an object with properties that include the list of paths to be clipped (CLIP.subject), the list of paths to clip with (CLIP.clip) and the type of clipping to perform (CLIP.type).  CLIP has the following properties:
* CLIP.subject - CLIP.subject is a list of paths.  Each path within the list is a list of points of the form {X: x, Y: y}. CLIP.subject contains the subject of the clipping operation.  In the code example above, the subject was the star because we wanted to perform clipping operations on the star using the circle.  CLIP.subject is required and must be a defined object, but it is allowed to be empty.  In other words, it is allowed to be a list containing a single empty list like [[]].  This is because the union operation (clipper.ClipType.ctUnion) does not distinguish between the subject and clip and simply adds everything together in both the subject and the clip, and it is often easier to put all paths into either the subject or the clip for this operation.
* CLIP.clip - CLIP.clip is a list of paths.  Each path within the list is a list of points of the form {X; x, Y; y}.  CLIP.clip contains the paths that will be used to clip the subject.  In the code example above, the circle was the clip because we wanted to perform clipping operations on the star using the circle.  CLIP.clip is required and must be a defined object, but it is allowed to be empty.  In other words, is allowed to be a list containing a single empty list like [[]].  This is because the union operation (clipper.ClipType.ctUnion) does not distinguish between the subject and clip and simply adds everything together in both the subject and the clip, and it is often easier to put all paths into either the subject or the clip for this operation.
* CLIP.type - CLIP.type is string that specifies the type of clipping operation to perform.  It can be one of the following strings:
  * "INTERSECTION" - If CLIP.type is equal to "INTERSECTION" the resulting paths will form the intersection of the subject and the clip. In the example above the leftmost cut is the intersection of the star with the circle.  Notice that only the area that is inside both polygons is left to from the star with the points cut off.
  * "UNION" - If CLIP.type is equal to "UNION", the resulting paths will include any area that was any of the polygons.  In the example above, the union of the star and circle was formed and the result is shown in the second cut.
  * "DIFFERENCE" - If CLIP.type is equal to "DIFFERENCE", the resulting paths will be the areas in the subject polygons minus the areas in the clip polygons.  In the example, the third image is the difference and shows only the points of the star because the circle was subtracted from the star.
  * "XOR" - If CLIP.type is equal to "XOR", only the areas that fall in either the subject or the clip, but not both will remain in the result.  This is shown in the fourth image in the example.  The fourth image looks like a circle and a star, but it is actually ten polygons that make up the opposite of the first image.

####Results
clip(CLIP) will return 0 if no errors are detected or throw an error message if an error is detected.  The following properties are loaded into the argument object, CLIP if no error is detected.
* CLIP.solutions - If no error is detected, CLIP.solutions will be loaded with the list of paths that result from the clipping operation.  CLIP.solutions is a list of paths.  Each of the paths within the list of paths is a list of point objects in the form of {X: x, Y: y}.

####Error Messages
If an error is detected, clip(CLIP) throws one of the following error messages:
* "ARGUMENT\_NOT\_PROVIDED" - The clip operation could not run because no argument (CLIP) was provided.
* "INVALID\_ARGUMENT" - The clip operationn could not run because the arugment that was provided was not an object.
* "SUBJECT\_NOT\_PROVIDED" - The clip operation could not run because the subject (CLIP.subject) was not provided.  At a minimum, the subject must be an empty list of lists.
* "INVALID\_SUBJECT" - The clip operation could not run because the CLIP.subject that was provided was not an object"
* "CLIP\_NOT\_PROVIDED" - The clip operation could not run because the clip (CLIP.clip) was not provided.  At a minimum, the subject must be and empty list of lists.
* "INVALID\_CLIP" - The clip operation could not run because the CLIP.clip property that was provided was not an object.
* CLIP\_TYPE\_NOT\_PROVIDED" - The clip operation could not run because the CLIP.type property was not provided.
* "INVALID\_CLIP\_TYPE" - The clip operation could not run because the CLIP.type property that was provided was not one of the following strings:
  * "INTERSECTION"
  * "UNION"
  * "DIFFERENCE"
  * "XOR"

[Back to Table of Contents](#Table of Contents)

###offset(OFF) <a name = 'offset' />
####Description
offset(OFF) provides a list of paths that are traces along existing polygons.  The traces can either be inside or outside of the existing polygons. The amount that the traces are offset from the existing polygons is given by an offset value.  Offset values that are negative will trace inside the existing polygons and offsets that are positive will trace along the outside of polygons by the offset value.

[Back to Table of Contents](#Table of Contents)

####Example
The following code shows some of the power of the Clipper Library.  It starts by making a rectangle and an ellipse and moving the ellipse to the right using functions in the Drawing Aids Library. It then creates another rectangle and places it over the first rectangle and the ellipse.  Next, it uses clip(CLIP) with the first rectangle and the ellipse as the subject and the second rectangle as the clip and gets the difference.  Next, it repetitively calls offset(OFF) with larger and larger negative offsets on the solutions from the difference clip.  Each time offset(OFF) is called it returns a set of solutions that are cut paths.  Those paths are cut on each succession until offset(OFF) no longer returns any solutions.  The result of this cutting is shown below.

It's worth noting that it starts by cutting single paths along the inner border of the of the first rectangle and the ellipse, and it cuts along the outer border of the second rectangle.  These two cutting patterns continue and "squeeze in" on one-another until a single path cannot be created without crossing other paths.  At that point, offset is able to divide the paths into separate paths, providing more solutions but not wasting cutting patterns over areas that have already been cut.

As you can see, this is a great way to cut out very complex pocket shapes.

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
r.width = 100,r.height = 100;
da.makeRectangle(r);
var e = {width: 150,height:100,increments: 100};
da.makeEllipse(e);
var m = {x: 100,y: 0,paths: [e.ellipse]}
da.moveBy(m);

var r2 = {width: 150, height: 50}
da.makeRectangle(r2);
var m2 = {x: 50, y: 0, paths: [r2.rect]};
da.moveBy(m2);

var CLIP = {subject: [m.newPaths[0],r.rect],clip: m2.newPaths,type: "DIFFERENCE"};
ca.clip(CLIP);

var OFF = {};
OFF.paths = CLIP.solutions;
OFF.offset = 0;

do {
	ca.offset(OFF);
	for(var i = 0;i<OFF.solutions.length;i++) cutter.cutPath(OFF.solutions[i],safeHeight,depth);
	OFF.offset -= bitWidth/2;
} while (OFF.solutions.length > 0);
```


The resulting cuts, which were simulated in the [Camotics](http://openscam.org) simulator are shown here.  Note that the cutting paths are displayed.  Close observation of those paths will give some clues as to how the resulting list of paths in each successive set of solutions look.

<img src = "https://github.com/buildbotics/tpl-docs/blob/master/images/offsetexample.png" height="300" width = "400">

####Arguments
offset(OFF) accepts a single argument, OFF.  OFF is an object with properties that include the list of paths to be offset, the method of joining polygons together, the end type of the polygons, and the offset to use.  OFF has the following properties:
* OFF.paths - OFF.paths is required and is a list of paths.  Each path within the list is a list of points in the form of {X: x, Y: y}.  This is the list of paths whose borders will be traced at an offset given by OFF.offset.
* OFF.offset - OFF.offset is a number that specifies the amount that the traces will be offset from the original polygon paths.  Positive OFF.offset values will cause the solutions to be outside the outer polygons and negative OFF.offset values will cause the solutions to be inside the outer polygons.  
* OFF.joinType - OFF.joinType is optional.  When provided, it is a string that specifies how the polygons in OFF.paths will be joined to together.  The default is "MITER".  Please refer to the [clipper.js](http://sourceforge.net/p/jsclipper/wiki/documentation/#clipperlibjointype) documentation for further explanation of joinTypes.  The following values are acceptable:
  * "MITER" - This is the default.
  * "ROUND"
  * "SQUARE"
* OFF.endType - OFF.endType is a string that specifies how the line ends will be shaped.  OFF.endType is optional and the default value  is "CLOSEDPOLYGON".  Please refer to the [clipper.js](http://sourceforge.net/p/jsclipper/wiki/documentation/#clipperlibendtype) documentation for further explanation of joinTypes. The following values are possible:
  * "OPENSQUARE"
  * "OPENROUND"
  * "OPENBUTT"
  * "CLOSEDLINE"
  * "CLOSEDPOLYGON"
  
Note, this clipping library is meant to work with closed polygons.  You will find that neither clip(CLIP) nor offset(OFF) will return open polygons, and the first and last points in each path returning from those functions are the same.  As a result, joinType and endType are mostly irrelavent.  You are welcome to experiment with them and let us know what you find.  However, those wishing to use such features are encouraged to use the [clipper.js](http://sourceforge.net/projects/jsclipper/) library directly.

###Results
####Results
offset(OFF) will return 0 if no errors are detected or throw an error if an error is detected.  The following properties are loaded into the argument object, CLIP if no error is detected.
* OFF.solutions - If no error is detected, OFF.solutions will be loaded with the list of paths that result from the offset operation.  OFF.solutions is a list of paths.  Each of the paths within the list of paths is a list of point objects in the form of {X: x, Y: y}.

####Error Messages
If an error is detected, offset(OFF) throws one of the following error messages:
* "ARGUMENT\_NOT\_PROVIDED" - The offset operation could not run because no argument object (OFF) was not provided.
* "INVALID\_ARGUMENT" - The offset operation could not run because the argment was not an object.
* "PATHS\_NOT\_PROVIDED" - The offset operation could not run because a list of paths to be offset was not provided.
* "PATHS\_INVALID" - The offset operation could not run because the list of paths that was provided was not in the form of a list of lists of points in the form of {X: x, Y: y}.
* "OFFSET\_NOT\_PROVIDED" - The offset operation could not run because the offset was not provided.
* "OFFSET\_INVALID" - The offset operation could not run because the OFF.offset that was provided was not a number.
* "JOIN\_TYPE\_INVALID" - The offset operation could not run because the OFF.joinType value that was provided was not one of the valid strings for OFF.joinType.
* "END\_TYPE\_INVALID" - The offset operation could not run because the OFF.endType value that was provided was not one of the valid strings for OFF.joinType.

[Back to Table of Contents](#Table of Contents)

###makePathsPolygons(P) <a name = 'makePathsPolygons' />
####Description
makePathsPolygons(P) was created as a helper function that eases the use of open ended strokes with the Clipper Library.  It's purpose is to take a list of open-ended paths, and trace around them to create very thin polygons.  These resulting polygons can then be handed to offset(OFF) or clip(CLIP) to to perform their functions.  makePathsPolygons(P) accepts a single object (P) as an argument, which has members that include the list of paths to convert to polygons and the distance that resulting traces will be outward from the original path.

[Back to Table of Contents](#Table of Contents)

####Example
This example is not a typical use of makePathsPolygons(P), but is intended to show its affect on a set of paths.  The code below starts by retrieving the paths that represent the string "Buildbotics" using the getLineOfText() function in the [HersheyText Aids Library](https://github.com/buildbotics/tpl-docs/blob/master/HersheyText%20Aids%20Library.md).  The paths for the string in "Sans 1-Stroke" font are returned in document.paths.  It then cuts the result, which is shown in the top part of the [Camotics](http://openscam.org) simulator image below in the single line fonts.  It then uses makePathsPolygons(P) to turn the open-ended paths into polygons.  The middle part of the [Camotics](http://openscam.org) simulator image below shows the resulting cut.  Notice that each of the original paths is now a closed polygon.  Finally, clip(CLIP) is called to create a "UNION" of the polygons to make single, continuous polygons, which is shown in the bottom part of the [Camotics](http://openscam.org) simulator image.

Once again, the author does not recommend this method of managing single line fonts.  A better scheme would be to create very thin polygons by setting P.delta to a small number (i.e. .01), and then using offset(OFF) with OFF.offset set to the 1/2 the width of the polygons that you actually want.

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
var hf = require('hersheytext');
var ha = require('HersheyTextAids');
var da = require('DrawingAids');
var cutter = require('CuttingAids');

var document = {};
document.text = 				"Buildbotics";
document.spacing = 			-1;
document.scale = 				5;
document.lineHeight = 	40;
document.font = 				"Sans 1-stroke";
document.spaceSize = 		5;
document.tabInterval = 	40;
ha.getLineOfText(document);
translate(0,400,0);
for (var i = 0; i < document.paths.length; i++ ) cutter.cutPath(document.paths[i],safeHeight,depth);

translate(0,-200,0);
var P = {paths: document.paths, delta: 5};
ca.makePathsPolygons(P);
for (var i = 0; i < P.polys.length; i++ ) cutter.cutPath(P.polys[i],safeHeight,depth);

translate(0,-200,0);
var C = {subject: P.polys, clip: [[]], type: "UNION"};
ca.clip(C);
for (var i = 0; i < C.solutions.length; i++ ) cutter.cutPath(C.solutions[i],safeHeight,depth);
```

The resulting cuts, which were simulated in the [Camotics](http://openscam.org) simulator are shown here.

<img src = "https://github.com/buildbotics/tpl-docs/blob/master/images/makepathspolygons.png" height="300" width = "400">

####Arguments
makePathsPolygons(P) accepts a single argument, P.  P is an object with properties that include the list of paths to convert to polygons and the distance outward that the new polygons will be from the original paths.  P has the following properties:
* P.paths - P.paths is required and is a list of paths.  Each path within the list is a list of points in the form of {X: x, Y: y}.  This is the list of paths that will be traced at a an outward distance given by P.delta.
* P.delta - P.delta is required and must be a positive number.  P.delta specifies the distance that the traces around the P.paths will be. P.delta will be 1/2 the width of the new polygons.

####Results
makePathsPolygons(P) will return 0 if no errors or throw an error message if an error is detected.  The following properties are loaded into the argument object, P if no error is detected.
* P.polygons - P.polygons is a list of paths.  Each path within the list is a list of points in the form of {X: x, Y: y}. P.polygons will be contain one polygon for each path in P.paths, and that polygon will be a path that traverses around the polygon at a distance of P.delta.

####Error Messages
If an error is detected, makePathsPolygons(P) throws one of the following error messages:
* "ARGUMENT\_NOT_\PROVIDED" - Polygons could not be created because the single object argument (P) was not provided.
* "ARGUMENT\_INVALID" - Polygons could not be created because the argument (P) was not an object.
* "PATHS\_NOT\_PROVIDED" - Polygons could not be created because P.paths was not defined.
* "PATHS\_INVALID" - Polygons could not be created because P.paths was not in the form of a list of paths, with each path being a list of points in the form of {X; x,Y: y};
* "DELTA\_NOT\_PROVIDED" - Polygons could not be created because P.delta was not defined.
* "DELTA\_INVALID" - Polygons could not be created because P.delta was not a number.
* "DELTA\_NOT\_POSITIVE" - Polygons could not be created because P.delta was less than or equal to zero.

[Back to Table of Contents](#Table of Contents)

