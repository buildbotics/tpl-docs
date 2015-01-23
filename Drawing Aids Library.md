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
DrawingAids functions all use a similar style, which is described as follows:
*Functions accept and return points in object form.  For instance a point at x = 1 and y =2 would be given as {X: 1, Y: 2}.  The only exception is makePointsObjects() which is provided to convert points in the form of [x,y] to the correct object form.
*Each function accepts a single argument which is an object.  The required properties differ among the various functions.
*Functions return a value of 0 if successful and -1 if not successful.
*Functions returning -1 add a property called "error" to the argument object which provides a description of the reason for failing.
*Function returning 0 add a property that includes the result to the argument object.  The resulting property name differs among functions.

