#Cutting Library
##Overview
The purpose of the Cutting Library is to provide some commonly used functions needed for cutting, thereby simplifying the task of writing [tplang](http://tplang.org) programs.  The Cutting Library makes use of the [Clipping Library'(https://github.com/buildbotics/tpl-docs/blob/master/Clipping%20Library.md).

## Style
The Cutting Library functions all use a similar style, which is described as follows:
* Functions accept and return points in object form.  For instance a point at x = 1 and y = 2 would be given as {X: 1, Y: 2}.
* Each function accepts a single argument which is an object.  The required properties differ among the various functions.
* Functions return a value of 0 if successful.
* If and error is detected functions throw an error string which suggests the problem encountered.  The message and stack trace can be read by catching the error an variable and the reading the stack property of that error.
* Functions returning 0 add properties that include the result to the argument object.  The resulting property names differ among functions.

##Functions
###cutPath(P)
####Description
cutPath(P) accepts a single object (P) as an argument, which contains the path to cut, the safe height for rapid moves, and the cutting depth.  With this information cutPath(P) outputs the [g-code](http:reprap.org/wiki/G-code) needed to cut the path.

####Example
The following code creates a 100 by 75 rectangle and cuts it out.
```
units(METRIC); // units are in inches
feed(30); // feed rate us 30 inches per minute
speed(4000); // spindle speed is 4000 rpm
var bitWidth = 3.125;
var safeHeight = 3;
var depth = 6.4;
tool(1);

var ca = require('ClipperAids');
var da = require('DrawingAids');
var cutter = require('CuttingAids');

/*
test cutPath
*/
try {
	var r = {width: 100,height:75};
	da.makeRectangle(r);
	var c = {path: r.rect, safeHeight: safeHeight, depth: depth}
	cutter.cutPath(c);
} catch (err) {
	console.log(err.stack);
};
```
The resulting [g-code](http:reprap.org/wiki/G-code), which was simulated in the [Camotics](http://openscam.org) simulator is shown here.

<img src = "https://github.com/buildbotics/tpl-docs/blob/master/images/simplerectangle.png" height="300" width = "400">

####Arguments
cutPath(P) accepts a single argument, P.  P is an object with properties that includes a path to be cut (P.path), the safe height for rapid movement (P.safeHeight), and the cutting depth (P.Depth).  P has the following properties:
* P.path - P.path is a list of points that are the path to cut.  cutPath(P) will raise the cutting head to the height specified in P.safeHeight, then move to the first point in P.paths, then lower the cutting head to the depth specified in P.depth, and then cut to each point in P.path.  When the last point is reached, cutPath(P) raises the cutting head to P.safeHeight.  Each point in P.path must be in the form of {X: x, Y: y}.
* P.safeHeight - P.safeHeight is a number that specifies the height that the cutting head will be raised to for rapid moves.
* P.depth - P.depth is a number that specifies the depth that the cutting head will be lowered to for cutting.  Positive values of depth will cause the head to drop below the surface of the workpiece.

####Results
cutPath(P) will return 0 if successful and will throw and error if an error is detected.  cutPath(P) does not set an properties in the P argument object.

####Error Messages
If and error is detected, cutPath(p) will throw one of the following error messages:
* "ARGUMENT\_NOT\_PROVIDED - G-code could not be created because the argument object (P) was not provided.
* "INVALID\_OBJECT" - G-code could not be created because the argument that was provided was not an object.
* "PATH\_NOT\_PROVIDED" - G-code could not be created because path (P.path) was not provided.
* "PATH\_INVALID" - G-code could not be created because the path was not a list of points in the form of {X: x, Y: y};
* "SAFE\_HEIGHT\_NOT\_PROVIDED" - G-code could not be created because the safe height for rapid movements (P.safeHeight) was not provided.
* "INVALID\_SAFE\_HEIGHT" - G-code could not be created because the value provided for P.safeHeight was not an number.
* "CUTTING\_DEPTH\_NOT\_PROVIDED" - G-code could not be created because the cutting depth (P.depth) was not provided.
* "INVALID\_CUTTING\_DEPTH" - G-code could not be created because the cutting depth (P.depth) that was provided was not a number.


