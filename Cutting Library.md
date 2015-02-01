#Cutting Library
##Overview
The purpose of the Cutting Library is to provide some commonly used functions needed for cutting, thereby simplifying the task of writing [tplang](http://tplang.org) programs.  The Cutting Library makes use of the [Clipping Library](https://github.com/buildbotics/tpl-docs/blob/master/Clipping%20Library.md).

## Style
The Cutting Library functions all use a similar style, which is described as follows:
* Functions accept and return points in object form.  For instance a point at x = 1 and y = 2 would be given as {X: 1, Y: 2}.
* Each function accepts a single argument which is an object.  The required properties differ among the various functions.
* Functions return a value of 0 if successful.
* If an error is detected, functions throw an error string which suggests the problem encountered.  The message and stack trace can be read by catching the error variable and the reading the stack property of that error (e.g. err.stack).
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
cutPath(P) will return 0 if successful and will throw an error if an error is detected.  cutPath(P) does not set any properties in the P argument object.

####Error Messages
If an error is detected, cutPath(P) will throw one of the following error messages:
* "ARGUMENT\_NOT\_PROVIDED - G-code could not be created because the argument object (P) was not provided.
* "INVALID\_ARGUMENT" - G-code could not be created because the argument that was provided was not an object.
* "PATH\_NOT\_PROVIDED" - G-code could not be created because the path (P.path) was not provided.
* "PATH\_INVALID" - G-code could not be created because the path was not a list of points in the form of {X: x, Y: y};
* "SAFE\_HEIGHT\_NOT\_PROVIDED" - G-code could not be created because the safe height for rapid movements (P.safeHeight) was not provided.
* "INVALID\_SAFE\_HEIGHT" - G-code could not be created because the value provided for P.safeHeight was not a number.
* "CUTTING\_DEPTH\_NOT\_PROVIDED" - G-code could not be created because the cutting depth (P.depth) was not provided.
* "INVALID\_CUTTING\_DEPTH" - G-code could not be created because the cutting depth (P.depth) that was provided was not a number.

###cutPanel(PANEL)
####Description
cutPanel(PANEL) accepts a single argument, containing properties that include a list of polygons, the rapid movement safe height, the cutting depth, the bit width, an offset value, and optionally a limit on the number of concentric loops to cut.  Using these properties, cutPanel(PANEL) cuts out the areas defined by the list of polygons.

Note - For reliable results, the paths in all polygons provided in the PANEL.shapes parameter must travel in the same direction.  The results will be unpredictable if some paths travel clockwise and others travel counter-clockwise.

####Example
The following code creates a rectangle and an eclipse as shapes and then cuts out the entire area inside those shapes.
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

try {
	var r = {width: 100,height:75};
	da.makeRectangle(r);
	var e = {width: 75, height:120,increments:100}
	da.makeEllipse(e)
	var p = {shapes: \[r.rect,e.ellipse\],
					 safeHeight: safeHeight,
					 depth: depth,
					 offset: -bitWidth/2,
					 bitWidth: bitWidth};
	cutter.cutPanel(p);
} catch (err) {
	console.log(err.stack);
};
```
The resulting [g-code](http:reprap.org/wiki/G-code) was simulated in the [Camotics](http://openscam.org) simulator and the result is shown here.

<img src = "https://github.com/buildbotics/tpl-docs/blob/master/images/cutpanel.png" height="300" width = "400">

####Arguments
cutPanel(PANEL) accepts a single argument, PANEL.  PANEL is an object with properties that include a list of polygons to be cut (PANEL.shapes), the safe height for rapid movement (PANEL.safeHeight), the cutting depth (PANEL.depth), the bit width (PANEL.bitWidth), the distance from the edge of shapes where cutting will start (PANEL.offset), and optionally a limit on the number of concentric cuts that will occur (PANEL.loops).  PANEL has the following properties:
* PANEL.shapes - PANEL.shapes is a list of paths.  Each path is a polygon and is a list of points in the form {X: x, Y: y}.  For reliable results, each of the polygon paths must travel in the same direction.  The results will be unpredictable if some of the polygon paths travel clockwise while others travel counter-clockwise.
* PANEL.safeHeight - PANEL.safeHeight is a number that specifies the height at which the cutting head will travel during rapid movements (i.e. when not cutting).
* PANEL.depth - PANEL.depth is a number that specifies the cutting depth.  Positive values cause the cutting depth be go below the surface of the workpiece material.
* PANEL.bitWidth - PANEL.bitWidth is a positive number that specifies the width of the bit that will be used.  It will be the distance between each concentric cut during cutting.  It is sometimes appropriate to specify values that are less than the actual bit width in order to ensure that all material is cut away.  If the results in the [Camotics](http://openscam.org) simulator show that some material is left uncut, try decreasing the magnitude of PANEL.bitWidth.
* PANEL.offset - PANEL.offset is a number that specifies the distance from the mask and boundary borders where cutting will begin.  Positive values of PANEL.offset will cause cutting path to be outside the boundary and encroach on the mask.  Negative values do the opposite.  It is often appropriate to set PANEL.offset to - 1/2 of the bitWidth.  This causes the first cutting path to be 1/2 the bit width inside the boundary and outside the mask and results in cutting that just touches the boundary and masks.  Sometimes this strategy will leave uncut material in sharp corners.  This can be partially remedied by increasing PANEL.offset.
* PANEL.loops - PANEL.loops is an optional value and is a number that allows limiting the number of concentric loops that will be taken before returning.  PANEL.loops should be a positive number.  If it is a negative number or not defined, all unmasked area inside the boundary is cut away.  PANEL.loops is useful for trimming the edges, sometimes with a smaller bit.

####Results
cutPanel(PANEL) will return 0 if successful and will throw an error if an error is detected.  cutPanel(PANEL) does not set any properties in the PANEL argument object.

####Error Messages
If an error is detected, cutPanel(PANEL) will throw one of the following error messages:
* "ARGUMENT\_NOT\_PROVIDED" - The panel could not be cut because the argument object (PANEL) was not provided.
* "INVALID\_ARGUMENT" - The panel could not be cut because the argument was not an object.
* "SHAPES\_NOT\_PROVIDED" - The panel could not be cut because the PANEL.shapes property was not defined.
* "SHAPES\_INVALID" - The panel could not be cut because the PANEL.shapes property was not in the form of a list of paths with each path being a list of points in the form of {X; x, Y: y}.
* "SAFE\_HEIGHT\_NOT\_PROVIDED" - The panel could not be cut because because the rapid movement height PANEL.safeHeight was not provided.
* "INVAID\_SAFE\_HEIGHT" - The panel could not be cut because the rapid movement property PANEL.safeHeight was not a number;
* "CUTTING\_DEPTH\_NOT\_PROVIDED" - The panel could not be cut because the cutting depth property PANEL.depth was not provided.
* "CUTTING\_DEPTH\_INVALID" - The panel could not be cut because the cutting depth property PANEL.depth that was provided was not a number.
* "BIT\_WIDTH\_NOT\_PROVIDED" - The panel could not be cut because the bit width property PANEL.bitWidth was not provided.
* "BIT\_WIDTH\_INVALID" - The panel could not be cut because the bit width property PANEL.bitWidth that was provided was not a number.
* "OFFSET\_NOT\_PROVIDED" - The panel could not be cut because the offset property PANEL.offset was not provided.
* "INVALID\_OFFSET" - The panel could not be cut because the offset property PANEL.offset that was provided was not a number.
* "LOOPS\_INVALID" - The panel could not be cut because the loops property (PANEL.loops) that was provided was not a number.

###emboss(EMB)
####Description
emboss(EMB) accepts a single argument, containing properties that include a boundary polygon (EMB.boundary), a set of masks (EMB.masks), the rapid movement head height ((EMB.safeHeight), the cutting depth (EMB.depth), the width of the cutting bit (EMB.bitWidth), the offset from the from the polygon border where the cutting will begin (EMB.offset), and optionally, the number of cutting loops to make (EMB.loops).  Using these properties, emboss(EMB) cuts out the areas inside the boundary but leaving the masks intact.  The main purpose of emboss(EMB) allow uncut areas to project outward from the surface of the material.
####Example
The following code gets paths for the string "Buildbotics" in Sans 1-stroke font and converts them to polygons.  Then it creates an ellipse that is large enough to encompass the text and moves it to a location where it surrounds the text.  Finally, it calls emboss to cut the areas not enclosed by the text polygons.

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

var P = {paths: document.paths, delta: .01};
ca.makePathsPolygons(P);

var E = {width: document.width + 200,
				 height: document.height + 300,
				 increments: 300}
da.makeEllipse(E);
var M = {paths: [E.ellipse],
				 x: document.width / 2,
				 y: document.height / 2}
da.moveBy(M);
var EMB = {boundary: M.newPaths[0],
					 masks: P.polys,
					 safeHeight: safeHeight,
					 depth: depth,
					 bitWidth: bitWidth,
					 offset: -9};
cutter.emboss(EMB);
```
The resulting [g-code](http:reprap.org/wiki/G-code) was simulated in the [Camotics](http://openscam.org) simulator and the result is shown here.

<img src = "https://github.com/buildbotics/tpl-docs/blob/master/images/embossbbinellipse.png" height="300" width = "400">

####Arguments
emboss(EMB) accepts a single argument, EMB.  EMB is an object with properties that includes a polygon to be used as a boundary (EMB.boundary), a list of polygons to be used as a mask (EMB.masks), the safe height for rapid movement (EMB.safeHeight), the cutting depth (EMB.depth), the bit width (EMB.bitWidth), the distance from the edges of the mask and the edge of the boundary where cutting will begin (EMB.offset), and optionally the number of concentric loops to cut before stopping (EMB.loops).  EMB has the following properties:
* EMB.boundary - EMB.boundary is a list of points in the form of {X; x, Y: y} that form a polygon.  It is used as the boundary around the areas that will be cut away.
* EMB.masks - EMB.masks is a list of polygons, with each polygon being a path that is a list of points in the form of {X; x, Y: y}.  EMB.masks form the mask and specify the areas inside the boundary that will not be cut away.
* EMB.safeHeight - EMB.safeHeight is a number that specifies the height at which the cutting head will travel during rapid movements (i.e. when not cutting).
* EMB.depth - EMB.depth is a number that specifies the cutting depth.  Positive values cause the cutting depth be go below the surface of the workpiece material.
* EMB.bitWidth - EMB.bitWidth is a positive number that specifies the width of the bit that will be used for cutting.  It will be the distance between each concentric cut.  It is sometimes appropriate to specify values that are less than the actual bit width in order to cause all material to be cut away.  If the results in the [Camotics](http://openscam.org) simulator show that some material is left uncut, try decreasing the magnitude of EMB.bitWidth.
* EMB.offset - EMB.offset is a number that specifies the distance from the mask and boundary borders where cutting will begin.  Positive values of EMB.offset will cause the cutting paths to be outside the boundary and encroach on the mask.  Negative values do the opposite.  It is often appropriate to set EMB.offset to - 1/2 of the bitWidth.  This causes the first cutting path to be 1/2 the bit width inside the boundary and outside the masks and results in cutting that just touches the boundary and masks.  Sometimes this strategy will leave uncut material in sharp corners.  This can be partially remedied by increasing EMB.offset.
* EMB.loops - EMB.loops is an optional value and is a number that allows limiting the number of concentric loops that will be taken before returning.  EMB.loops should be a positive number.  If it is a negative number or not defined, all unmasked area inside the boundary is cut away.  EMB.loops is useful for trimming the edges, sometimes with a smaller bit.

####Results
emboss(EMB) will return 0 if successful and will throw an error if an error is detected.  emboss(EMB) does not set any properties in the EMB argument object.

####Error Messages
If an error is detected, emboss(EMB) will throw one of the following error messages:
* "ARGUMENT\_NOT\_PROVIDED" - emboss(EMB) could not execute because the argument object (EMB) was not provided.
* "ARGUMENT\_INVALID" - emboss(EMB) could not execute because the argument that was provided was not an object.
* "BOUNDARY\_NOT\_PROVIDED" - emboss(EMB) could not execute because the boundary polygon (EMB.boundary) was not provided.
* "BOUNDARY\_INVALID" - emboss(EMB) could not execute because the boundary that was provided was not in the form of a list of points with each point being in the form of {X: x, Y: y}.
* "MASKS\_NOT\_PROVIDED" - emboss(EMB) could not execute because the list of masks (EMB.masks) was not provided.
* "MASKS\_INVALID" - emboss(EMB) could not execute because the EMB.masks property that was provided was not in the form of a list of paths, with each path being a list of points in the form of {X: x, Y: y}.
* "SAFE\_HEIGHT\_NOT\_PROVIDED" - emboss(EMB) could not execute because the safe height property (EMB.safeHeight) was not provided.
* "SAFE\_HEIGHT\_INVALID" - emboss(EMB) could not execute because the safe height property (EMB.safeHeight) that was provided was not a number.
* "CUTTING\_DEPTH\_NOT\_PROVIDED" - emboss(EMB) could not execute because the cutting depth property (EMB.depth) was not provided.
* "CUTTING\_DEPTH\_INVALID" - emboss(EMB) could not execute because the cutting depth property (EMB.depth) that was provided was not a number.
* "BIT\_WIDTH\_NOT\_PROVIDED" - emboss(EMB) could not execute because the bit width property (EMB.bitWidth) was not provided.
* "BIT\_WIDTH\_INVALID" - emboss(EMB) could not execute because the bit width property (EMB.bitWidth) that was provided was not a number.
* "OFFSET\_NOT\_PROVIDED" - emboss(EMB) could not execute because the offset property (EMB.offset) was not provided.
* "INVALID\_OFFSET" - emboss(EMB) could not execute because the offset property (EMB.offset) that was provided was not a number.
* "LOOPS\_INVALID" - emboss(EMB) could not execute because the loops property (EMB.loops) that was provided was not a number.


