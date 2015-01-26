# DrawingAids Library
## Overview
The DrawingAids library is meant to be used with [tplang](http://tplang.org), and provides a set of commonly needed drawing tools.
## Example
This example shows how the DrawingAids Library can be used in conjunction with the CuttingAids library to create the [g-code](http:reprap.org/wiki/G-code) needed to cut out a eight-point star with rounded vertexes.

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
try {
	da.makeStar(star);
	cutter.cutPath(star.star,safeHeight,depth);
} catch(err) {
	print(err,'\n');
};
```
The resulting cuts, which were simulated in the [Cambotics](http://openscam.org) simulator are shown here.

<img src = "https://github.com/buildbotics/tpl-docs/blob/master/8PointStar.png" height="300" width = "400">

## Style
DrawingAids functions (except for makePointsObjects() ) all use a similar style, which is described as follows:
* Functions accept and return points in object form.  For instance a point at x = 1 and y = 2 would be given as {X: 1, Y: 2}.
* Each function accepts a single argument which is an object.  The required properties differ among the various functions.
* Functions return a value of 0 if successful and -1 if not successful.
* Functions returning -1 throw an error string with suggests the problem encountered.
* Functions returning 0 add a property that includes the result to the argument object.  The resulting property name differs among functions.

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
bezierPoint(BEZ) accepts an object argument (BEZ) that is loaded with a four-point list and a position argument.  It returns the point found at "position' along the bezier curve described in the four-point list.
####Example
The following code captures points at 10% intervals along a bezier curve and then cuts them out.
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
bezierPoint(BEZ) accepts a single argument (BEZ) which is an object containing the following members.
* BEZ.position - Position is a fraction between zero and one that denotes the percentage of the distance from the first point, points[0] to the last point, points[3] where the desired point is to be retrieved.
* BEZ.points = Points is the list of four points that define the bezier curve.


####Results
bezierPoint(BEZ)_ always returns 0. In addition, it sets the following member value:
* BEZ.result - contains that point along the bezier curve described by the points in "points" at the postion described by "position".

####Error Messages
bezierPoint(BEZ) does not provide any error checking at this time.

###FourPointTensionedBSpline(S)
####Description
FourPointTensionedBSpline(S) accepts one object argument, S, and creates a spline curve between  the two "knots" in the point list provided in "S.points" and places the result in the "S.spline" property of the argument object, S.  The tension of the resulting spline is defined by the S.tension property and the number of points to return is defined by the S.grain property.
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
FourPointTensionedBSpline(S) accepts a single argument, S, that must be preloaded with the following properties:
* S.points - contains a list of four points.  The spline will be created that represents the curve between the two knots which are the second and third points in points.
* S.tension - S.tension describes how closely the curve will adhere to the points.  Higher values of tension tend to cause the curve to pull away from the points causing the general shape to be lost and distorted.  The image above shows the results with tension = 4.  More reasonable values of tension are between .1 and 1.5.
* S.grain - S.grain specifies how many points to return to approximate the curve.

####Results
FourPointTensionedBSpline(S) always returns 0 and will place the resulting curve in the spline property of the argument object.
* S.spline - contains that points generated.

####Error Messages
FourPointTensionedBSpline(S) does not provide any error checking at this time.

### makeSpline(S)
####Description
makeSpline(S) accepts an object that contains an extended list of points provided in S.points and places a spline curve that approximates the list while smoothing out corners in S.spline.  The extent to which the curve pulls away from the points given depends on the tension parameter provided in S.tension.  The granularity of the curves is specified in S.grain.
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
S.points = da.makePointsObjects([[15,40],[15,35],[20,30],[20,20],[17,20],
																 [17,0],[25,0],[25,20],[23,20],[23,30],
																 [18,35],[18,40],[15,40]]);
S.tension = 1;
S.grain = 10;
try {
	da.makeSpline(S)
	cutter.cutPath(S.points,safeHeight,depth);
	translate(50,0,0);
	cutter.cutPath(S.spline,safeHeight,depth);
} catch (err) {
	print(err,'\n');
};
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
* S.spline - If no errors are detected, a list of points representing the resulting spline is placed in S.spline.

####Error Messages
If an error is detected the following error will be trown.
* "INSUFFICIENT\_NUMBER\_OF\_POINTS"

###polyhedron(P)
####Description
polyhedron(P) accepts a single object (P) as an argument and creates a polyhedron based on the content of the object.  The contents of P will contain the number of sides to the polyhedron and the radius of the polyhedron.  Optionally, P may specify that the vertexes of the polyhedron be rounded and if so how many increments should be provided in each rounded vertex.  The resulting polyhedron is centered around the origin {X: 0,Y: 0}
####Example
The following code creates a five point polyhedron with a radius of 100 and rounded vertexes.  The radius of the vertexes is 15 and each rounded vertex is broken into 10 increments.
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

var p = {};
p.radius = 100, p.count = 5, p.cradius = 15, p.cincs = 10;
try {
	da.polyhedron(p);
	cutter.cutPath(p.polyhedron,safeHeight,depth);
} catch (err) {
	print(err,'\n');
};
```
The following image shows the [Cambotics](http://openscam.org) simulation of the resulting [g-code](http:reprap.org/wiki/G-code).

<img src = "https://github.com/buildbotics/tpl-docs/blob/master/polyhedron.png" height="300" width = "400">
####Arguments
polyhedron(P) accepts a single argument.  That argument (P) is an object that contains the following properties:
* P.radius - P.radius is the distance from the center (located at {X: 0, Y: 0}) to each vertex.  Note that the vertexes are not reached if they are rounded.  Therefore, the radius will actually be greater than the distance from the origin to the farthest reaching points in the rounded polyhedron.
* P.count - P.count specifies the number of sides (or vertexes) contained in the polyhedron.
* P.cradius - P.cradius is an optional argument that specifies the radius of the rounded vertexes.
* P.cincs - P.cincs is only required if P.cradius is defined and greater than 0.  P.cincs specifies the number of increments that make up each rounded corner.  Large values of cincs create smoother cuts, but generate more [g-code](http:reprap.org/wiki/G-code).

####Results
polyhedron(P) returns 0 if successful and -1 if an error is detected.  The following properties are set in the argument object (P):
* P.polyhedron - P.polyhedron contains the list of points that make up the resulting polyhedron.  P.polyhedron will only be defined if no error is detected during execution and 0 is returned.

####Error Messages
If an error is detected, polyhedron(P) returns -1 and throws  one of the following values:
* "RADIUS\_OF\_POLYHEDRON\_NOT\_PROVIDED" - P.radius was not defined.  The radius is required to define the size of the polygon.
* "RADIUS\_INVALID\_TYPE" - P.radius was provided, but is not a number.  A number is required to specify the radius of the polygon.
* "COUNT\_NOT\_PROVIDED" - P.count was not defined. P.count specifies the number of sides on the polygon and is required to define the shape of the polygon.
* "COUNT\_IS\_NOT\_A\_NUMBER" - P.count was provided but is not a number.  The number of sides of the polygon must be be a number.
* "COUNT\_IS\_LESS\_THAN\_THREE" - P.count was provided, but is less than three.  Polygons require at least three sides.
* "CRADIUS\_INVALID\_TYPE" - P.cradius was provided, but it is not a number.  The corner radius must be a number.
* "CINCS\_NOT\_DEFINED" - P.cradius was provided, but P.cincs was not defined.  The number of increments to make up a corner must be specified if the corner radius is greater than zero.
* "CINCS\_INVALID\_TYPE" - P.cincs was provided, but it is not a number.  The corner radius must be a number.
* "CINCS\_IS\_LESS\_THAN\_2" - P.cincs is less than two.  Less than two increments would not round the polygon vertexes.

###makeArc(A)
####Description
makeArc(A) accepts a single object as an argument and creates an arc.  The object contains the starting point for the arc, the center point of the arc, the number of increments (line segments) that make up the arc, and the angle (in radians) over which the arc will be drawn.
####Example
The following code creates an arc starting at {X: 0, Y: 0} around the center point at {X:0,Y:50} extending over pi radians (1/2 circle) and consisting of 100 line segments.
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

var A = {};
A.start = {X:0,Y:0}, A.center = {X:0,Y:50},A.increments = 100,A.angle = Math.PI;
try {
	da.makeArc(A);
	cutter.cutPath(A.arc,safeHeight,bitWidth);
} catch (err) { print(err,'\n'); };
```
The following image shows the [Cambotics](http://openscam.org) simulation of the resulting [g-code](http:reprap.org/wiki/G-code).

<img src = "https://github.com/buildbotics/tpl-docs/blob/master/arc.png" height="300" width = "400">

####Arguments
makeArc(A) accepts a single argument.  That argument (A) is an object that contains the following properties:
* A.start - A.start is an object containing a point in the form of {X: x, Y: y}.  This is the point where the arc will begin.
* A.center - A.center is an object containing a point in the form of {X: x, Y: y}.  The is the center point around which the arc will be drawn.
* A.increments - A.increments specifies the number of increments that form the arc.  Each increment will ultimately be drawn as a line segment.  Larger values of A.increments will create smoother arcs, but will cause the resulting  [g-code](http:reprap.org/wiki/G-code) file to be larger.
* A.angle - A.angle is the angle over which the resulting arc will be struck.  The value is given in radians.  Positive values cause the arc to be struck counter-clockwise and negative values cause the arc to be struck clockwise.

####Results
makeArc(A) returns 0 if no errors are detected and -1 if an error is detected.  The following properties will be set in the argument object (A) depending on whether errors are detected:
* A.arc - If no error is detected, A.arc contains a list of points that form the resulting arc.  The points will be in the form of {X: x, Y: y}.

####Error Messages
If an error is detected, makeArc(A) returns -1 and throws one of the following values:
* "STARTING\_POINT\_NOT\_DEFINED" - An arc could not be struck because the starting point is not defined.
* "INVALID\_STARTING\_POINT" - An arc could not be struck because the starting point is not an object.
* "CENTER\_POINT\_NOT\_DEFINED" - An arc could not be struck because the center point is not defined.
* "INVALID\_CENTER\_POINT" - An arc could not be struck because the center point not an object.
* "NUMBER\_OF\_INCREMENTS\_NOT\_DEFINED" - An arc could not be drawn because the number of increments is not defined.
* "NUMBER\_OF\_INCREMENTS_INVALID" - An arc could not be drawn because the number of increments is not a number.
* "ANGLE\_NOT\_DEFINED" - An arc could not be struck because the angle was not defined.
* "INVALID\_ANGLE" - An arc could not be struck because the angle is not a number.

###makeRectangle(R)
####Description
makeRectangle(R) creates rectangles.  The rectangles can have square or rounded corners.  It accepts a single argument, which is an object containing the desired width and height, and optionally the radius and number of increments in rounded corners.  The resulting rectangle is centered around the origin {X: 0,Y: 0}.

####Example
The following code creates a rectangle with a width of 150 and a height of 100.  The rectangle has rounded corners with corner radii of 10.  Each corner will be broken into 10 increments (10 line segments). 
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

r = {};
r.width = 150, r.height = 100, r.cornerRadius = 10, r.cornerIncrements = 10;
try { 
	da.makeRectangle(r);
	cutter.cutPath(r.rect,safeHeight,bitWidth);
} catch (err) { print(err,'\n'); };
```
The following image shows the [Cambotics](http://openscam.org) simulation of the resulting [g-code](http:reprap.org/wiki/G-code).

<img src = "https://github.com/buildbotics/tpl-docs/blob/master/rect.png" height="300" width = "400">

####Arguments
makeRectangle(R) accepts a single argment, R.  R is an object with properties that describe the characteristics of the desired rectangle.  R has the following properties:
* R.width - R.width is the width of the rectangle.  It is required and must be a number.
* R.height - R.height is the height of the rectangle.  It is required and must be a number.
* R.cornerRadius - R.cornerRadius denotes the radius of the rectangle corners.  It is optional and if not provided, it will be assumed to be zero.  R.cornerRadius must be a number.
* R.cornerIncrements - R.cornerIncrements must be provided if R.cornerRadius is greater than 0.  R.cornerIncrements is the number of line segments that each corner will be broken into.  It must be a number.  Larger values of R.cornerIncrements provide smoother edges but cause the resulting g-code files to be larger.

####Results
makeRectangle(R) will return 0 if no errors are detected and -1 if an error is detected.  The following properties are loaded into the argument object, R depending on whether an error is detected.
* R.rect - R.rect is a list of points in the form of {X: x, Y: y} that form the desired rectangle.  R.rect will remain unchanged (undefined if it hasn't been proviously set) if an error is detected.  If no error is detected, R.rect will be loaded with the resulting rectangle.

####Error Messages
If an error is detected, makeRectangle(R) returns -1 and throws one of the following values:
* "HEIGHT\_NOT\_DEFINED" - A rectangle could not be formed because the height was not provided.
* "INVALID\_HEIGHT" - A rectangle could not be formed because the height was provided but was not a number.
* "WIDTH\_NOT\_DEFINED" - A rectangle could not be formed because the width was not provided.
* "INVALID\_WIDTH" - A rectangle could not be formed because the width was provided but was not a number.
* "INVALID\_CORNER\_RADIUS" - A rectangle was not formed because the cornerRadius that was provided was not a number.
* "CORNER\_INCREMENTS\_NOT\_DEFINED" - A rectangle was not formed because the corner radius was provided and was greater than 0, but the number of corner increments was not specified.
* "CORNER\_INCREMENTS\_INVALID" - A rectangle was not formed because the corner radius was provided and was greater than 0, but the specified number of corner increments was not a number.

###makeStar(S)
####Description
makeStar(S) creates a star.  The resulting star can have rounded inner and outer vertexes.  It accepts a single argument, which is an object containing the desired radius at the other vertexes, the radius at the inner vertexes, the number of points on the desired star, and optionally the radius and number of increments in rounded outer and inner vertexes.  The radius of the outer and inner vertexes may be different and are specified independently. The resulting star is centered around the origin {X: 0,Y: 0}.

####Example
The following code creates an 8-pointed star with an outer vertex radius of 100, an inner vertex radius of 30.  The star has rounded inner and outer vertexes.  The outer vertexes are rounded around a radius of 6 and broken into 10 increments (10 line segments).  The inner vertexes are rounded around a radius of 3 and broken into 5 increments. 
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
try {
	da.makeStar(star);
	cutter.cutPath(star.star,safeHeight,depth);
} catch(err) {
	print(err,'\n');
};
```
The following image shows the [Cambotics](http://openscam.org) simulation of the resulting [g-code](http:reprap.org/wiki/G-code).

<img src = "https://github.com/buildbotics/tpl-docs/blob/master/star.png" height="300" width = "400">

####Arguments
makeStar(S) accepts a single argument, S.  S is an object with properties that describe the characteristics of the desired star.  S has the following properties:
* S.outerRadius - S.outerRadius is a number that specifies the radius at the points of the outer vertexes.  Note that the actual radius of the star points will be less if the points are rounded.
* S.numberOfPoints - S.numberOfPoints is a number that specifies the number of points in the star.
* S.innerRadius - S.innerRadius is a number that specifies the radius at the inner vertexes.  Note that the inner radius will not be reached if the inner vertexes are rounded
* S.radiusOfOuterVertexes - S.radiusOfOuterVertexes is optional and, if provided, is a number that specifies the radius around which the outer vertexes will be rounded.
* S.outerVertexIncrements - S.outerVertexIncrements is only required if S.radiusOfOuterVertexes is greater than 0.  It is a number that specifies the number of line segments that the rounded outer vertexes will be broken into.  Larger values of S.outerVertexIncrements result in smoother cuts, but also generate larger [g-code](http:reprap.org/wiki/G-code) files.
* S.radiusOfInnerVertexes - S.radiusOfInnerVertexes is optional and, if provided, is a number that specifies the radius around which the inner vertexes will be rounded.
* S.innerVertexIncrements - S.innerVertexIncrements is only required if S.radiusOfInnerVertexes is greater than 0.  It is a number that specifies the number of line segments that the rounded inner vertexes will be broken into.  Larger values of S.innerVertexIncrements result in smoother cuts, but also generate larger [g-code](http:reprap.org/wiki/G-code) files.

####Results
makeStar(S) will return 0 if no errors are detected and -1 if an error is detected.  The following properties are loaded into the argument object, S depending on whether an error is detected.
* S.star - S.star is a list of points in the form of {X: x, Y: y} that form the desired star.  S.star will remain unchanged (undefined if it hasn't been proviously set) if an error is detected.  If no error is detected, S.star will be loaded with the resulting star.

####Error Messages
If an error is detected, makeStar(S) returns -1 and throws one of the following values:
* "STAR\_OBJECT\_ARG\_NOT\_DEFINED" - A star could not be created because no argument was provided.
* "STAR\_INVALID\_TYPE" - A star could not be created because an argument was provided but it was not an object.
* "OUTER\_RADIUS\_ARGUMENT\_NOT\_PROVIDED" - A star could not be created because the outer radius was not provided.
* "OUTER\_RADIUS\_ARGUMENT\_INVALID\_TYPE" - A star could not be created because the outer radius was provided but it was not a number.
* "NUMBER\_OF\_POINTS\_NOT\_PROVIDED" - A star could not be created because the number of points was not provided. 
* "NUMBER\_OF\_POINTS\_INVALID\_TYPE" - A star could not be created because the number of points property was not a number.
* "NUMBER\_OF\_POINTS\_NOT\_AN\_INTEGER" - A star could not be created because the number of points specified was not an integer.
* "INSUFFICIENT\_NUMBER\_OF\_POINTS" - A star could not be created because the number of points specified was less than three.
* "INNER\_RADIUS\_NOT\_PROVIDED" - A star could not be created because the inner radius was not provided.
* "INNER\_RADIUS\_INVALID\_TYPE" - A star could not be created because the inner radius that was provided was not a number.
* "INNER\_RADIUS\_TOO\_BIG" - A star could not be created because the inner radius was greater than the outer radius.
* "RADIUS\_OF\_OUTER\_VERTEXES\_INVALID\_TYPE" - A star could not be created because the radius of the outer vertexes that was provided was not a number.
* "OUTER\_VERTEX\_INCREMENTS\_NOT\_DEFINED" - A star could not be created because the radius of outer vertexes was greater than 0, but the number of outer vertex increments was not provided.
* "OUTER\_VERTEX\_INCREMENTS\_INVALID_TYPE" - A star could not be created because the radius of outer vertexes was greater than 0, but the number of outer vertex increments was not a number
* "OUTER\_VERTEX\_INCREMENTS\_NOT\_AN\_INTEGER" - A star could not be created because the radius of outer vertexes was greater than zero, but the number of increments was not an integer.
* "RADIUS\_OF\_INNER\_VERTEXES\_INVALID\_TYPE" - A star could not be created because the number of inner vertexes that was provided was not a number.
* "INNER\_VERTEX\_INCREMENTS\_NOT\_DEFINED" - A star could not be created because the radius of inner vertexes was specified, but the number of inner vertex increments was not.
* "INNER\_VERTEX\_INCREMENTS\_INVALID\_TYPE" - A star could not be created because the radius of inner vertexes was specified, but the number of inner vertexes that was specified was not a number.
* "INNER\_VERTEX\_INCREMENTS\_NOT\_AN\_INTEGER" - A star could not be created because the radius of inner vertexes was specified, but the number of inner vertexes that was specified was not an integer.

###makeEllipse(E)

####Description
makeEllipse(E) creates an ellipse.  It accepts a single argument, which is an object containing the width, height, and number of line segments of the desired ellipse.  The resulting ellipse will be centered around the origin (X: 0,Y: 0}.

####Example
The following code creates an ellipse that is 70 points wide, 200 points high and consists of 100 line segments.
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

try {
	var E = {};
	E.width = 70, E.height = 200, E.increments = 100;
	da.makeEllipse(E);
	cutter.cutPath(E.ellipse,safeHeight,depth);
} catch (err) {
	print(err,'\n');
};
```
The following image shows the [Cambotics](http://openscam.org) simulation of the resulting [g-code](http:reprap.org/wiki/G-code).

<img src = "https://github.com/buildbotics/tpl-docs/blob/master/ellipse.png" height="300" width = "400">

####Arguments
makeEllipse(E) accepts a single argument, E.  E is an object with properties that describe the characteristics of the desired ellipse.  E has the following properties:
* E.width - E.width is a number that specifies the width of the desired ellipse.
* E.height - E.height is a number that specifies the height of the desired ellipse.
* E.increments - E.increments is a number that specifies the number of line segments that will make up the desired ellipse.  Larger values for E.increments will create smoother cuts, but will generate larger [g-code](http:reprap.org/wiki/G-code) files.

####Results
makeEllipse(E) will return 0 if no errors are detected and -1 if an error is detected.  The following properties are loaded into the argument object, E depending on whether an error is detected.
* E.ellipse - E.ellipse is a list of points in the form of {X: x, Y: y} that form the desired ellipse.  E.ellipse will remain unchanged (undefined if it hasn't been proviously set) if an error is detected.  If no error is detected, E.ellipse will be loaded with the resulting ellipse.

####Error Messages
If an error is detected, makeEllipse(E) will throw one of the following error messages.
* "ELLIPSE\_OBJECT\_ARG\_NOT\_DEFINED" - An ellipse could not be created because no argument was provided to makeEllipse(E).
* "ELLIPSE\_OBJECT\_INVALID\_TYPE" - An ellipse could not be created because the argument that was provided was not an object.
* "ELLIPSE\_WIDTH\_NOT\_DEFINED" - An ellipse could not be created because the width of the ellipse was not provided.
* "ELLIPSE\_WIDTH\_NOT\_VALID" - An ellipse could not be created because the width of the ellipse that was provided was not a number.
* "ELLIPSE\_HEIGHT\_NOT\_DEFINED" - An ellipse could not be created because the height of the desired ellipse was not provided.
* "ELLIPSE\_HEIGHT\_INVALID" - An ellipse could not be created because the height that was provided was not a number.
* "ELLIPSE\_INCREMENTS\_NOT\_DEFINED" - An ellipse could not be created because the desired number of increments that would make up the resulting ellipse was not provided.
* "ELLIPSE\_INCREMENTS\_NOT\_VALID" - An ellipse could not be created because the desired number of increments that would make up the resulting ellipse was not a number.

###moveBy(M)
####Description
moveBy(M) moves an existing list of paths to a new location.  It accepts a single argument, which is an object containing the list of paths (M.paths) to be moved and the x and y coordinates to move by.  The new list of paths will be identical to the original list of paths except it will be transposed by M.x points along the x-axis and M.y points along the y axis.

####Example
The following code creates and cuts a five-point, rounded polygon, then moves it 100 points along the x axis and 100 points along the y axis and cuts it again.
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

var P = {};
P.radius = 100, P.count = 5, P.cradius = 15, P.cincs = 10;
try {
	da.polyhedron(P);
	cutter.cutPath(P.polyhedron,safeHeight,depth);
	var M = {};
	M.paths = [P.polyhedron], M.x = 100, M.y = 100;
	da.moveBy(M);
	cutter.cutPath(M.newPaths[0],safeHeight,depth);
} catch (err) { 
	print(err,'\n');
};
```
The following image shows the [Cambotics](http://openscam.org) simulation of the resulting [g-code](http:reprap.org/wiki/G-code).

<img src = "https://github.com/buildbotics/tpl-docs/blob/master/movedemo.png" height="300" width = "400">

####Arguments
moveBy(M) accepts a single argument, M.  M is an object with properties that include the set of paths to be moved and the amount to move them in the x and y directions.  M has the following properties:
* M.paths - M.paths is a list of paths.  Each path in the list is a list of points of the form {X: x, Y: y}.  The paths in M.paths will be moved by M.x points in the x direction and M.y points in the y direction.
* M.x - M.x is an optional argument and is a number that specifies the number of points to move along the x axis.  If M.x is not defined, it is assumed to be 0;
* M.y - M.y is an optional argument and is a number that specifies the number of points to move along the y axis.  If M.y is not defined, it is assumed to be 0;

####Results
moveBy(M) will return 0 if no errors are detected and -1 if an error is detected.  The following properties are loaded into the argument object, M depending on whether an error is detected.
* M.newPaths - If no errors are detected, M.newPaths will contain a list of paths identical to M.paths except moved by x points along the x axis and y  points along the y axis.  M.newPaths will be a list of paths and each path within the list will be a list of points in the form {X: x, Y: y}.

####Error Messages
If an error is detected, moveBy(M) will throw one of the following error messages:
* "ARGUMENT\_OBJECT\_UNDEFINED" - The move could not be accomplished because no argument object was provided.
* "ARGUMENT\_OBJECT\_INVALID" - The desired move could not be accomplished because an argument was provided but it was not an object.
* "PATHS\_NOT\_PROVIDED" - The desired move could not be accomplished because the M.paths property was not defined.
* "PATHS\_NOT\_LIST\_OF\_LISTS\_OF\_POINTS" - The desired move could not be accomplished because the M.paths object was not a list of paths with points in the form of {X: x, Y: y}.
* "X\_MOVE\_VALUE\_INVALID" - The desired move could not be accomplished because the M.x property that was provided was not a number.	
* "Y\_MOVE\_VALUE\_INVALID" - The desired move could not be accomplished because the M.y property that was provided was not a number.

