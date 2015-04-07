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
