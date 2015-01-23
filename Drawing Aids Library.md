# DrawingAids Library
## Overview
The DrawingAids library is meant to be used with [tplang](http://tplang.org), and provides a set of commonly needed drawing tools.
## Example
This example shows how the DrawingAids Library can be used in conjunction with the CuttingAids library to create the [g-code](http://http://reprap.org/wiki/G-code) needed to cut out a six-point star with rounded vertexes.

'''
var star = {};
star.outerRadius = 500;
star.numberOfPoints = 8;
star.innerRadius = 50;
star.radiusOfOuterVertexes = 23;
star.outerVertexIncrements = 10;
star.radiusOfInnerVertexes = 5;
star.innerVertexIncrements = 5;
if(da.makeStar(star) != 0) print(star.error,'\n');
cutter.cutPath(star.path,safeHeight,depth);
'''

