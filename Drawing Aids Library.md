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
[8 Point Star](https://github.com/buildbotics/tpl-docs/8PointStar.png).
