#Testbed for exploring OpenVG on the Raspberry Pi.

The program "shapes" exercises a high-level API built on OpenVG. If you run the program 
with no command line arguments, you get a "reference card" that demonstrates the calls in the library. 
Running with a numeric command line argument shows the specified number of random shapes.

## API

Coordinates are VGfloat values, with the origin at the lower left, with x increasing to the right, and y increasing up.
Color is specified as a VGfloat array containing red, green, blue, alpha values ranging from 0.0 to 1.0.
Shapes are filled with the color specified as fill[4], stroke color as stroke[4], and stroke width as sw.

	void Start(int width, int height, float fill[4])
Begin the picture, clear the screen with the specified color

	void Line(VGfloat x1, VGfloat y1, VGfloat x2, VGfloat y2, 
				VGfloat sw, VGfloat stroke[4]))
draw a line between (x1, y1) and (x2, y2)

	void Rect(VGfloat x, VGfloat y, VGfloat w, VGfloat h, VGfloat sw, VGfloat fill[4], VGfloat stroke[4])
draw a rectangle with its origin (lower left) at (x,y), and size is (width,height)

	void Roundrect(VGfloat x, VGfloat y, VGfloat w, VGfloat h, VGfloat rw, VGfloat rh, VGfloat sw, 
					VGfloat fill[4], VGfloat stroke[4])
draw a rounded rectangle with its origin (lower left) at (x,y), and size is (width,height).  The width and height of the corners are specified with (rw,rh)

	void Poly(VGfloat *x, VGfloat *y, VGint n, VGfloat sw, 
			VGfloat fill[4], VGfloat stroke[4], VGboolean dofill)
draw a polyline (dofill=true), or polygon (dofill=false), using the coordinates in arrays points to by x and y.  The number of coordinates is n.

	void Circle(VGfloat x, VGfloat y, VGfloat r, VGfloat sw, 
				VGfloat fill[4], VGfloat stroke[4])
draw a circle centered at (x,y) with radius r.

	void Ellipse(VGfloat x, VGfloat y, VGfloat w, VGfloat h, 
				VGfloat sw, VGfloat fill[4], VGfloat stroke[4])
draw an ellipse centered at (x,y) with radii (w, h).

	void Qbezier(Gfloat sx, VGfloat sy, VGfloat cx, VGfloat cy, VGfloat px, VGfloat py, VGfloat ex, VGfloat ey, 
		VGfloat sw, VGfloat fill[4], VGfloat stroke[4]))
draw a quadratic bezier curve beginning at (sx, sy), using control points at (px, py), ending at (ex, ey)

	void Cbezier(VGfloat sx, VGfloat sy, VGfloat cx, VGfloat cy, VGfloat px, VGfloat py, VGfloat ex, VGfloat ey, 
		VGfloat sw, VGfloat fill[4], VGfloat stroke[4])
draw a cubic bezier curve beginning at (sx, sy), using control points at (cx, cy) and (px, py), ending at (ex, ey)

	void Arc(VGfloat x, VGfloat y, VGfloat w, VGfloat h, VGfloat sa, VGfloat aext, 
		VGfloat sw, VGfloat fill[4], VGfloat stroke[4])
draw an elliptical arc centered at (x, y), with width and height at (w, h).  Start angle (degrees) is sa, angle extent is aext.

	void Text(VGfloat x, VGfloat y, const char* s, int pointsize, 
				VGfloat fillcolor[4], VGPath *gp, const short *characterMap, const int *glyphAdvances, VGbitfield renderFlags)
draw a the text srtring (s) at location (x,y), using pointsize, filled with fillcolor. 

The font information: 
* gp is an array of glpyh paths
* characterMap,
* glyphAdvances

renderflags specifies how the text will be rendered.
	
	End()
end the picture, rendering to the screen.


## Using fonts

Also included is the font2openvg program, which will turn font information into C source that 
you can embed in your program. The Makefile makes font code from files found in /usr/share/fonts/truetype/ttf-dejavu/. 
If you want to use other fonts, adjust the Makefile accordingly, or generate them on your own once the font2openvg program is built.

font2openvg takes three arguments: the TrueType font file, the output file to be included and the prefix for identifiers.
For example to use the DejaVu Sans font:

	./font2openvg /usr/share/fonts/truetype/ttf-dejavu/DejaVuSans.ttf DejaVuSans.inc DejaVuSans

and include the generated code in your program:

	#include "DejaVuSans.inc"
	
	VGPath DejaVuSansPaths[256];
	
The loadfont function creates OpenVG paths from the font data:

	loadfont(DejaVuSans_glyphPoints, DejaVuSans_glyphPointIndices,
            DejaVuSans_glyphInstructions, DejaVuSans_glyphInstructionIndices,
            DejaVuSans_glyphInstructionCounts, DejaVuSans_glyphCount,
            DejaVuSansPaths)

The unloadfont function releases the path information:

	unloadfont(DejaVuSansPaths, DejaVuSans_glyphCount)

# Build and run

	pi@raspberrypi ~/vg $ make fonts shapes
	g++ -I /usr/include/freetype2 font2openvg.cpp   -o font2openvg -lfreetype
	for f in /usr/share/fonts/truetype/ttf-dejavu/*.ttf; do fn=`basename $f .ttf`; ./font2openvg $f $fn.inc $fn; done
	224 glyphs written
	224 glyphs written
	224 glyphs written
	224 glyphs written
	224 glyphs written
	224 glyphs written
	cc -Wall -I/opt/vc/include -I/opt/vc/include/interface/vcos/pthreads -o shapes shapes.c -L/opt/vc/lib -lGLESv2
	pi@raspberrypi ~/vg $ ./shapes # hit return when you are done looking at the awesomness

	pi@raspberrypi ~/vg $ ./shapes 100 # show 100 random shapes