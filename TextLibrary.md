#Text Library

## Table of Contents <a name = 'Table of Contents' />

| Section              | SubSection                                              |
|----------------------|:-------------------------------------------------------:|
|[Overview](#Overview) |                                                         |
|                      |[Style](#OverviewStyle)                                  |
|[Functions](#Functions)|                                                        |
|                      |[AddFontFolder(path)](#AddFontFolder)                    |
|                      |[ListAvailableFonts()](#ListAvailableFonts)              |
|                      |[UseFont(F)](#UseFont)                    |
|                      |[GetTextPaths(T)](#GetTextPaths)                    |

## Overview <a name = 'Overview' />

The Text Library provides simple access to a vast number of fonts.  It is able to read any scalable fonts that are accessible through the [FreeType2](http://www.freetype.org/) API.  In addition, it can read most of the fonts provided with the [LibreCAD](http://librecad.org/cms/home.html) 2D Computer Aided Design software.

The Text Library provides a wrapper around the [TPL](http://www.tplang.org) addFontDir(), listFonts(), and getCharCut() commands and simplifies their use.

The following example demonstrates how the Text Library is used.
```
var f = require("TextLibrary");
var clip = require("ClipperAids");
var cutter = require('CuttingAids');

units(IMPERIAL);

f.AddFontFolder("/usr/share/fonts");

var F1 = {fontName: "DejaVu Serif Condensed Italic", units: "inches", height: 3,
          characterSpacing: 3, wordSpacing: 7};
f.UseFont(F1);

var T = {text: "Hello World!"}
f.GetTextPaths(T);

var P = {safeHeight: .125, depth: .125}

for (var i = 0; i < T.paths.length; i++) {
  P.path = T.paths[i];
  cutter.cutPath(P);
}
```
The following image shows the [Camotics](http://openscam.org) simulation of the resulting [g-code](http:reprap.org/wiki/G-code).
<img src = "https://github.com/buildbotics/tpl-docs/blob/master/images/FontLibraryExample.png" height="300" width = "400">

[Back to Table of Contents](#Table of Contents)

## Style <a name = 'OverviewStyle' />
The Text Library style is described as follows:
* When cutting paths are returned, the points are in object form.  For instance a point at x = 1 and y = 2 would be given as {X: 1, Y: 2}.
* Functions requiring multiple arguments receive those arguments as properties in a single object.  The results are added as properties to that same object.
* Functions return a value of 0 if successful.
* If an error is detected functions throw an error message that suggests the problem that was encountered.  That message can be caught in the try-catch statement.  The error and the trace information can be accessed in the stack property of the err object (e.g. console.log(err.stack)).
* Functions returning 0 may add properties that include the result to the argument object.  The resulting property names differ among functions.

[Back to Table of Contents](#Table of Contents)

## Functions <a name = 'Functions' />
### AddFontFolder(path) <a name = 'AddFontFolder' />
#### Description
AddFontFolder provides a path to a folder where fonts are contained.  Upon receiving a path to a folder, AddFontFolder searches that folder and all subfolders for files containing fonts that are readable by GetTextPaths.  Each readable file is added to the list of known fonts and becomes available for use.

####Example
The following example adds all fonts in the /usr/share/librecad/fonts folder.
```
TEXT = require('TextLibrary');
TEXT.AddFontFolder("/usr/share/librecad/fonts");
```
When AddFontFolder encounters files that are not scalable or it does not recognize as a font file, it quietly ignores them.

####Results
0 is returned if no errors are encountered.

####Error Messages
If an error is encountered, AddFontFolder throws one of the following error messages:
* AddFontFolder: Font folder path not provided - The path argument was not provided.
* AddFontFolder: Invalid font folder path - The path argument was provided, but it was not a string.

[Back to Table of Contents](#Table of Contents)

### ListAvailableFonts() <a name = 'ListAvailableFonts' />
####Description
ListAvailableFonts() returns a list of all font types and associated font names that are currently available.  Fonts become available by providing paths to font folders using the AddFontFolder command.  ListAvailableFonts is typically used by TPL programmers to discover the names of available fonts, but is not typically used when generating g-code.

####Example
The following code demonstrates the use of ListAvailableFonts()
```
TEXT = require('TextLibrary');
TEXT.AddFontFolder("/usr/share/fonts");
var fontList = TEXT.ListAvailableFonts();
for (var i = 0; i < fonts.length; i++) {
  print(fonts[i].type,'\t',fonts[i].name,'\n');
};
```

####Results
ListAvailableFonts() returns an array of objects.  Each object in the array is of the form { type: type, name: name }.  The type property contains the file extension (e.g. ttf, lff, pfb).  The name property contains the name of the property that was gleaned from the font file.

[Back to Table of Contents](#Table of Contents)

###UseFont(F) <a name = 'UseFont' />
####Description
UseFont(F) accepts a single object argument, whose properties describe the font to be used in subsequent calls to GetTextPaths(T).  The Object members are:
* F.fontName - fontName is a string containing the name of the font to be used.  It must be a name of a font that has been previously discovered through AddFontFolders.  Valid font names can be found by using the ListAvailableFonts function.
* F.units - F.units is a string that specifies whether inches or millimeters will be used when scaling the font size.  It can contain one of the following four strings: "inches", "\"", "millimeters", or "mm".  The default value is "inches".
* F.height - F.height is a number that specifies the approximate height desired for the resulting fonts.  If units are in "inches", the default value is 1.  If the units are "mm", the default value is 25.4.
* F.characterSpacing - F.characherSpacing is a number that specifies the distance between characters.  Its value is assumed to be in the units specified in F.units and its default value is F.height / 3.
* F.wordSpacing - F.wordSpacing is a number that specifies the distance between words.  Its value is assumed to be in the units specified in F.units and its default value is equal to .75 * F.height.
* F.lineSpacingFactor - F.lineSpacingFactor is a number that specifies the distance between lines. It's value is multiplied by the F.height to determine the actual line spacing.  It's default value is .67.
* F.justify - F.justify specifies whether the text will be "LEFT", "CENTER", or "RIGHT" justified.  It is only relevant for multi-line text.  It's default value is "LEFT".

####Example
The following code is an example of using UseFont(F);
```
var f = require("TextLibrary");
units(IMPERIAL);

f.AddFontFolder("/usr/share/fonts");
var F1 = {fontName: "DejaVu Serif Condensed Italic", units: "inches", height: 3,
          characterSpacing: 3, wordSpacing: 7, lineSpacing: 2};
f.UseFont(F1);
```
This code specifies that text returned by GetTextPaths will use the "Dejavu Serif Condensed Italic" font, the characters will be 3 inches high and each character will be seperated by 3 inches.  Additionally, words will be separated by 7 inches and lines will be 6 inches apart.

####Results
If no errors are encountered, UseFont(F) simply returns 0.

####Error Messages
If an error is discovered, one of the following error messages will be thrown:
* UseFont: Argument not provided - No argument was provided
* UseFont: Invalid argument - The argument that was provided was not an object.
* UseFont: Font name not provided - No font name was provided.
* UseFont: Invalid font name - The font name that was provided was not a string.
* UseFont: Font not available - The font name that was provided was not found among the available fonts.
* UseFont: Invalid units - The value provided in the units member was not one of the following: "inches", "\"", "millimeters", or "mm".
* UseFont: Invalid character height - The height member that was provided was not a number.
* UseFont: character height must be greater than zero" - The height specified was less than or equal to zero.
* UseFont: Invalid character spacing - The character spacing that was provided was not a number.
* UseFont: Invalid word spacing - The word spacing that was provided was not a number.
* UseFont: Invalid line spacing - The line spacing factor that was provided was not a number.
* UseFont: justify string not recognized - The value string provided for justify was not "LEFT", "CENTER", or "RIGHT".

[Back to Table of Contents](#Table of Contents)

###GetTextPaths(T) <a name = 'GetTextPaths' />
####Description
GetTextPaths(T) accepts a single argument T that contains a text string and returns the paths required to cut the text.  GetTextPaths can accept strings ranging from a single character to multiple lines.  GetTextPaths uses the properties specified in the most recent UseFont(F) command to determine the font, character size, line spacing, word spacing and character spacing.  

####Examples
These examples assume that the UseFont(F) command was previously called and the Text Library has been loaded into f using the require command.

The following example shows how to use GetTextPaths(T) for a single character, 'R'.
```
var T = { text: "R"};
f.GetTextPaths(T);
```

The next example shows how to use GetTextPaths(T) for multiple lines of text.
```
var text = "Hello World!\n\
Goodbye World!";
var T = {text: text};
f.GetTextPaths(T);
```

####Results
If no errors are encountered GetTextPaths will return a list of paths representing the T.text.  The text will be positioned such that the left edge of the text will be at X=0 and the bottom of non-descending characters in the top line will be located at Y = 0.  The paths will be in the format specified in the most recent call to UseFont(F).

Each path in the list will be in the form of a list of points with each point being of the form {X: x, Y: y}.

####Error Messages
If an error is encountered, one of the following error messages will thrown:
* GetTextPaths: argument not provided - No argument was provided.
* GetTextPaths: invalid argument - The argument that was provided was not an object.
* GetTextPaths: no text was provided - The text string was not included in the argument object.
* GetTextPaths: text provided is not a character or string - The value that was provided in the text member was not a character or a string.
* GetTextPaths: Font is not defined - No font was defined before calling GetTextPaths(T).

[Back to Table of Contents](#Table of Contents)
