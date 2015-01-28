# HersheyText Aids Library
## Overview
The Hertext Aids Library (HertextTextAids.js) is provided to allow tplang to use the fonts library known as HersheyText.  HersheyText fonts were provided by the [Evil Mad Scientist Laboratories](http://www.evilmadscientist.com/2014/hershey-text-js/) in a file called HersheyText.js.  HersheyText.js is provided with the [Cambotics](http://openscam.org) distribution along with [tplang](http://tplang.org).

At present, HersheyText.js has only one fuction, which is getLineOfText(TC).  

## Style
HersheyTextAids.js style is described as follows:
* Functions accept and return points in object form.  For instance a point at x = 1 and y = 2 would be given as {X: 1, Y: 2}.
* Each function accepts a single argument which is an object.  The required properties differ among the various functions.
* Functions return a value of 0 if successful and -1 if not successful.
* Functions returning -1 throw an error string that suggests the problem encountered.
* Functions returning 0 add a resulting properties to the argument object.  The resulting property names differ among functions.

## Functions
### getLineOfText(TC)

