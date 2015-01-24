# DrawingAids Library
## Overview
The DrawingAids library is meant to be used with [tplang](http://tplang.org), and provides a set of commonly needed drawing tools.
## Example
This example shows how the DrawingAids Library can be used in conjunction with the CuttingAids library to create the [g-code](http:reprap.org/wiki/G-code) needed to cut out a six-point star with rounded vertexes.

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

var star = {};
star.outerRadius = 100;
star.numberOfPoints = 8;
star.innerRadius = 30;
star.radiusOfOuterVertexes = 6;
star.outerVertexIncrements = 10;
star.radiusOfInnerVertexes = 3;
star.innerVertexIncrements = 5;
if(da.makeStar(star) != 0) print(star.error,'\n');
cutter.cutPath(star.path,safeHeight,depth);
```
The resulting cuts, which were simulated in the [Cambotics](http://openscam.org) simulator are shown here.

<img src = "https://github.com/buildbotics/tpl-docs/blob/master/8PointStar.png" height="300" width = "400">

## Style
DrawingAids functions (except for makePointsObjects() ) all use a similar style, which is described as follows:
* Functions accept and return points in object form.  For instance a point at x = 1 and y = 2 would be given as {X: 1, Y: 2}.
* Each function accepts a single argument which is an object.  The required properties differ among the various functions.
* Functions return a value of 0 if successful and -1 if not successful.
* Functions returning -1 add a property called "error" to the argument object which provides a description of the reason for failing.
* Function returning 0 add a property that includes the result to the argument object.  The resulting property name differs among functions.

One exception to these style rules is found in makePointsObjects(), which is a helper function that is provided to easily convert point lists in the form of [x,y] to {X: x,Y: y}.  See the description of makePointsObjects() for more information.

## Functions
###makePointsObjects(pointList)
####Description
Converts a list of points in the form of [x,y] found in pointList to the form of {X: x,Y: y} and returns the resulting list.
####Arguments
pointList - a list of points.  The form of pointList is [[x1,y1],[x2,y2],...,[xn,yn]] .
####Results
returns a list of points in the form of [{X: x1,Y: y1},{X: x2,Y: y2},...,{{X: xn,Y: yn}] .
####Errors
no error checking is provided in makePointsObjects().  If it fails, you should review the format of the input list.
###bezierPoint(BEZ)
####Description
bezierPoint() accepts an object argument (BEZ) that is loaded with a four-point list and a position argument.  It returns the point found at "position' along the bezier curve described in the four-point list.
####Example
The following code returns the point that is 75% of the distance along the bezier curve described by the points in "points".
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

var bezier = {};
bezier.points = da.makePointsObjects([[0,0],[0,50],[70,30],[40,20]]);;

var path = [];
for (var i = 0; i <= 1; i+=.1) {
	bezier.position = i;
	da.bezierPoint(bezier);
	path.push(bezier.result);
}
cutter.cutPath(path,safeHeight,bitWidth);
```
The following image shows the [Cambotics](http://openscam.org) simulation of the resulting [g-code](http:reprap.org/wiki/G-code).
<img src = "https://github.com/buildbotics/tpl-docs/blob/master/BezierCurve.png" height="300" width = "400">

####Arguments
bezierPoint() accepts a single argument (BEZ) which is an object containing the following members.
* BEZ.position - Position is a fraction between zero and one that denotes the percentage of the distance from the first point, points[0] to the last point, points[3] where the desired point is to be retrieved.
* BEZ.points = Points is the list of four points that define the bezier curve.


####Results
bezierPoint always returns 0. In addition, it sets the following member value:
* BEZ.result - contains that point along the bezier curve described by the points in "points" at the postion described by "position".

###FourPointTensionedBSpline(S)
####Description
FourPointTensionedBSpline accepts one object argument, S, and creates a spline curve between  the two "knots" in the point list provided in "S.points" and places the result in the "S.spline" property of the argument object, S.  The tension of the resulting spline is defined by the S.tension property and the number of points to return is defined by the S.grain property.
####Example
The following code creates a 10 point spline between the two knots (points 1 and 2) of the point list.
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

S = {};
S.points = da.makePointsObjects([[0,0],[50,80],[70,200],[200,0]]);
S.tension = 4;
S.grain = 10;
da.FourPointTensionedBSpline(S);
cutter.cutPath(S.spline,safeHeight,depth);
```
The following image shows the [Cambotics](http://openscam.org) simulation of the resulting [g-code](http:reprap.org/wiki/G-code).
<img src = "https://github.com/buildbotics/tpl-docs/blob/master/4PointSpline.png" height="300" width = "400">
####Arguments
FourPointTensionedBSpline accepts a single argument, S, that must be preloaded with the following properties:
* S.points - contains a list of four points.  The spline will be created that represents the curve between the two knots which are the second and third point in points.
* S.tension - S.tension describes how closely the curve will adhere to the points.  Higher values of tension tend to cause the curve to pull away from the points causing the general shape to be lost and distorted.  The image above shows the results with tension = 4.  More reasonable values of tension are between .1 and 1.5.
* S.grain - S.grain specifies how many points to return to approximate the curve.

####Results
FourPointTensionedBSpline always returns 0 and will place the resulting curve in the spline property of the argument object.
* S.spline - contains that points generated.

### makeSpline(S)
####Description
makeSpline accepts an object that contains an extended list of points provided in S.points and places a spline curve that approximates the list while smoothing out corners in S.spline.  The extent to which the curve pulls away from the points given depends on the tension parameter provided in S.tension.  The granularity of the curves is specified in S.grain.
####Example
The following code creates an extended spline based on the points provided.  It then cuts the original points, translates to the right by 50 and cuts the spline.
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

S = {};
S.points = da.makePointsObjects([[15,40],[15,35],[20,30],[20,20],[17,20],										[17,0],[25,0],[25,20],[23,20],[23,30],
				[18,35],[18,40],[15,40]]);
S.tension = 1;
S.grain = 10;
da.makeSpline(S);
cutter.cutPath(S.points,safeHeight,depth);
translate(50,0,0);
cutter.cutPath(S.spline,safeHeight,depth);
```
The following image shows the [Cambotics](http://openscam.org) simulation of the resulting [g-code](http:reprap.org/wiki/G-code).  The cut on the left is the original point list while the cut on the right is the spline.

<img src = "https://github.com/buildbotics/tpl-docs/blob/master/ExtendedSpline.png" height="300" width = "400">

####Arguments
makeSpline(S) accepts a single object argument with the following properties:
* S.points - a list of points to be used as the basis for the spline.
* S.tension - a number that denotes how tight the curve will be.  Higher numbers make the spline "tighter" and pull farther away from the points.  Resonable values for tension are 1.5 and below.  The image above uses a tension value of 1.
* S.grain - specifies the number of increments between each two points.  Higher numbers produce smoother curves but result in larger [g-code](http:reprap.org/wiki/G-code) files.

####Results
makeSpline(S) returns 0 if successful and -1 if a failure is detected.
* S.spline - a list of points representing the resulting spline is placed in S.spline.
* S.error - S.error is only defined if an error is encountered.  If an error occurs (i.e. -1 is returned), then S.error is a string that contains the error message.

####Error Messages
S.error will contain one of the following error messages if -1 is returned.
* "INSUFFICIENT\_NUMBER\_OF\_POINTS"






