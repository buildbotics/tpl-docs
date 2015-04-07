#Font Library

## Table of Contents <a name = 'Table of Contents' />

| Section              | SubSection                                              |
|----------------------|:-------------------------------------------------------:|
|[Overview](#Overview) |                                                         |
|                      |[Style](#OverviewStyle)                                  |
|[Functions](#Functions)|                                                        |
|                      |[getLineOfText(TC)](#getLineOfText)                      |

## Overview <a name = 'Overview' />

The font library provides simple access to a vast number of fonts.  It is able to read any scalable fonts that are accessible through the [FreeType2](http://www.freetype.org/) API.  In addition, it can read most of the fonts provided with the [LibreCAD](http://librecad.org/cms/home.html) 2D Computer Aided Design software.

The font library provides a wrapper around the [TPL](http://www.tplang.org) addFontDir(), listFonts(), and getCharCut() commands and simplifies their use.

The following example shows a simple example of how the Font Library is used.
```
var f = require("TextLibrary");
var clip = require("ClipperAids");
var cutter = require('CuttingAids');

units(IMPERIAL);

f.AddFontFolder("/usr/share/librecad/fonts");
f.AddFontFolder("/usr/share/fonts");

var F1 = {fontName: "URW Chancery L Medium Italic", units: "inches", height: 1,
          characterSpacing: 3, wordSpacing: 7, lineSpacing: 20};
f.UseFont(F1);

var T = {text: "Hello World!", cursor: {X: 0, Y: 0}}
paths = f.GetTextPaths(T);

var P = {safeHeight: .125, depth: .25}

for (var i = 0; i < paths.length; i++) {
  P.path = paths[i];
  cutter.cutPath(P);
}
```
The following image shows the [Camotics](http://openscam.org) simulation of the resulting [g-code](http:reprap.org/wiki/G-code).
<img src = "https://github.com/buildbotics/tpl-docs/blob/master/images/FontLibraryExample.png" height="300" width = "400">

[Back to Table of Contents](#Table of Contents)

## Style <a name = 'OverviewStyle' />
The text library style is described as follows:
* When cutting paths are returned, the points are in object form.  For instance a point at x = 1 and y = 2 would be given as {X: 1, Y: 2}.
* Arguments are passed to functions as a single object.  The required properties differ among the various functions.
* Functions return a value of 0 if successful.
* If an error is detected functions throw an error message that suggests the problem that was encountered.  That message along with trace can be caught in the try-catch statement.  The error and the trace information can be accessed in the stack property of the err object (e.g. console.log(err.stack)).
* Functions returning 0 may add properties that includes the result to the argument object.  The resulting property names differ among functions.

[Back to Table of Contents](#Table of Contents)
