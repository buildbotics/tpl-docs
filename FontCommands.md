#API
##FontCommands
The font commands provide the ability to cut text in a multitude of fonts.

###addFontDir(path)
Adds a path to a directory or folder that contains font files. When issued, this command will look at all files in the named folder and all subfolders and determine whether the file contains a font that is compatible. The following types of fonts are known to be compatible:
* LibreCAD font files ending with a .lff file extension. Note, a few of the LibreCAD font files are not compatible because they do not contain character fields that are used to look up the character to be used in the font, but rather use four-hex-digit unicode values. Most LibreCAD fonts are single line fonts and there are a few outline fonts.
* Any fonts that the FreeType2 library is capable of querying for outlines. This includes most scalable fonts.

example: addFontDir("/usr/share/fonts");

return: An empty string is returned.

###listFonts()
listFonts returns a list of fonts that were found in the paths provided by calls to addFontDir. The list is in the form of a string that can be parsed with JSON.parse. listFonts provides the TPL programmer with a way to query TPL to get the names, types, and paths to each font file.
The fields provided for each known font are:
* "name" - This is the name of the font that was gleaned from the font file (e.g. "DejaVu Sans Bold").
* "type" - This is the file extension for the file containing the font (e.g. "ttf" or "lff").
* "path" - Contains the path name for the font file (e.g. "/usr/share/fonts/truetype/dejavu/DejaVuSans-Bold.ttf")
example: print(listFonts(),'\n');
return: Returns a string formatted as follows:
```
[
{ "name": "font name 1", "type": "aaa", "path": "pathname1"},
{ "name": "font name 2", "type": "bbb", "path": "pathname2"},
.
.
.
{ "name": "font name n", "type": "nnn", "path": "pathnamen"}
]
```

###getCharCuts(c,f)
getCharCuts accepts a one character string in c and a string containing the name of the font in f, which must be one of the known fonts found by calling addFontDir(path). It looks up the character in the font file providing the named font and returns a string that includes the cuts required to cut the character.
The return string is in JSON format and can be parsed using JSON.parse(). The return will be a list of objects describing each movement or cut. The possible cut or movement types are:
* { "type": "MOVE", "X": "x", "Y" "y"} - Rapid movement to point (x,y)
* { "type": "LINESEG", "X": "x", "Y" "y"} - Cut movement to point (x,y)
* { "type": "ARCSEG", "X": "x", "Y": "y", "angle": "a"} - Arcing cut starting at current point centered around point (x,y) by angle "a". If a is positive,  the cut is clockwise and if arc is negative the cut is counterclockwise.
* { "type": "CONIC", "X": x, "Y": "y", "CX": "cx", "CX": "cy"} - Cut from current point to (x,y) using a quadratic bezier curve with control point (cx, cy). The equation for the quadratic bezier curve is:
```
X = (1-t)*(1-t) * lastX + 2 * t * (1-t) * (cuts[i].CX) + t*t*(cuts[i].X);
Y = (1-t)*(1-t) * lastY + 2 * t * (1-t) * (cuts[i].CY) + t*t*(cuts[i].Y);
where t is an increment along the path between the current point (lastX,lastY) and the end point (x,y).
```
* { "type": "CUBIC", "X": x, "Y": "y", C1X", "cx1", "C1Y": "cy1", "C2X": "cx2", "C2Y": "cy2"} - Cut from current point to (x,y) using a cubic bezier curve through control points (cx1,cy1) and (cx2,cy2). The equation for the cubic bezier curve is:
```
X = (1-t)*(1-t)*(1-t)*lastX +
(1-t)*(1-t)*3*t*(cuts[i].C1X) +
t * t *(1 - t)*3*(cuts[i].C2X) +
t * t * t * (cuts[i].X);
Y = (1-t)*(1-t)*(1-t)*lastY +
(1-t)*(1-t)*3*t*(cuts[i].C1Y) +
t * t *(1 - t)*3*(cuts[i].C2Y) +
t * t * t * (cuts[i].Y);
where t is an increment along the path between the current point (lastX,lastY) and the end point (x,y).
```

example: cuts = JSON.parse(getChar("C","DejaVu Sans Bold");
