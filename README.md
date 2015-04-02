# // Barcode Writer in Pure JavaScript

Welcome to the new home for bwip-js.  The Google Code Project is shutting down so
this is where you will find the latest updates.


bwip-js is a translation to native JavaScript of the amazing code provided in [Barcode Writer in Pure PostScript](https://github.com/bwipp/postscriptbarcode).  The translated code can run on any browser that natively supports the HTML canvas element or any JavaScript-based server framework that can implement a minimal bitmap graphics interface.

* Current BWIPP version is 2015-03-24.
* Current version of bwip-js is 0.9.

bwip-js links:

* [Home page](http://metafloor.github.io/bwip-js/)
* [Repository](https://github.com/metafloor/bwip-js)
* [Online Barcode Generator](http://metafloor.github.io/bwip-js/demo/demo.html)
* [BWIPP documentation](https://github.com/bwipp/postscriptbarcode/wiki)
* [Symbologies supported by BWIPP/bwip-js](https://github.com/metafloor/bwip-js/wiki/Supported-Barcode-Symbologies)

## New in bwip-js

Scalable fonts, finally!  An
[Emscripten compiled](http://kripken.github.io/emscripten-site/index.html)
version of the [FreeType library](http://freetype.org/) is now integrated into bwip-js.

> **Version 0.8 is an API-breaking release.**  If you have implemented code based on 
> previous versions, you must update your code in three areas.  This 
> [wiki page](https://github.com/metafloor/bwip-js/wiki/v0.8-API-Changes) describes
> the changes.

Embedded in the FreeType library are open-source versions of the OCR-A and OCR-B
fonts provided by the
[Tsukurimashou Font Family](http://en.sourceforge.jp/projects/tsukurimashou/releases/)
project.

A description of how FreeType was compiled for use with bwip-js can be found at:
[Compiling FreeType with Emscripten](https://github.com/metafloor/bwip-js/wiki/Compiling-FreeType).

## Online Barcode Generator

An [online barcode generator](http://metafloor.github.io/bwip-js/demo/demo.html)
demonstrates all of the features of bwip-js.
It showcases the new font rendering provided by the FreeType library,
and allows using your own fonts.

The demo is tested on the latest versions of Firefox and Chrome, along with IE10.
IE11+ should work, and so should the latest versions of Opera and Safari,
but they are untested.


## Features

The barcode images generated by bwip-js are essentially identical to using BWIPP with 
[Ghostscript](http://www.ghostscript.com/).  The most significant
difference is the use of the OCR-A and OCR-B fonts as the defaults in bwip-js.
Barcodes and the OCR fonts are like
chocolate and hazelnut; they were meant to go together.  Unfortunately, most
PostScript environments do not provide the OCR fonts and must fallback to Courier,
Helvetica, and other less-than-ideal typefaces.

The following two images show the differences due to typeface.
The image below was rendered by bwip-js:

![bwip-js ISBN](http://metafloor.github.io/bwip-js/images/bwip-js-isbn.png?raw=true)

And the next image was rendered using BWIPP with Ghostscript:

![BWIPP/Ghostscript ISBN](http://metafloor.github.io/bwip-js/images/bwipp-isbn.png?raw=true)


The new font functionality is implemented with the following logic:

 * OCR-B is used as the default font for all barcodes.
 * OCR-A is used for the extra text on the ISBN, ISMN and ISSN symbols.

These defaults can be overridden using BWIPP options.
The fonts are known to the PostScript emulation as `OCR-A` and `OCR-B`.
For example, to switch the font to OCR-A, you would specify the option:

```
textfont=OCR-A
```

For the text above the barcode on the ISBN, ISMN, and ISSN symbols, the font
can be changed using ``isbntextfont``, ``ismntextfont``, and ``issntextfont``,
respectively.

The code in `demo.html` shows how to load your own fonts into bwip-js.  See 
[Preloading Fonts Into FreeType](https://github.com/metafloor/bwip-js/wiki/Preloading-Fonts-Into-FreeType)
and [Loading Fonts During Runtime](https://github.com/metafloor/bwip-js/wiki/Loading-Fonts-During-Runtime)
for descriptions of the two techniques.

The online barcode generator pre-loads the
[Inconsolata font](http://www.levien.com/type/myfonts/inconsolata.html),
which can be seen using the option:

```
textfont=Inconsolata
```

For example, we can render an alternate version of the ISBN barcodes shown above
with the Inconsolata font using the BWIPP options:

```
includetext guardwhitespace textfont=Inconsolata isbntextfont=Inconsolata isbntextsize=11
```

![ISBN Using Inconsolata](http://metafloor.github.io/bwip-js/images/isbn-inconsolata.png?raw=true)

## Installation

You can download the latest package at:

https://github.com/metafloor/bwip-js/releases/latest

Unzip the download package.  bwip-js will be unpacked into the following directory structure:

	bwip-js/
		bwip.js			# Main bwip-js module
		freetype.js		# The Emscripten-compiled FreeType library
		freetype.js.mem	# Demand loaded memory image
		demo.html		# The bwip-js demo
		node-demo		# An example node-js HTTP server application
		node-bwipjs		# A node-js module that implements bwip-js
		bwipp/			# The cross-compiled BWIPP encoders and renderers
		lib/			# Utilities required by the demo


## Usage

> **Using the demo with a file:// URL is no longer supported.**
>
> Emscripten optimizes the FreeType library by separating the memory intialization
> image and using XHR (when running in a browser) to demand load it.  Because of
> this, you can no longer reliably use the demo via the `file://` scheme in a
> browser's URL bar.  Firefox supports XHR with `file://`, Chrome and IE do not.

bwip-js was designed to run in both client-side and server-side JavaScript environments.  For server-side environments that do not provide a graphics API, there are freely available pure-JavaScript implementations of image file formats like PNG and BMP.  A quick search of the web turns up several possibilities that can be adapted for server-side usage. 

Using bwip-js in your code involves the following steps:

* Load the FreeType library.  The library module must load before the bwip-js module.
* Load the main bwip-js module.
* Implement two host-specific interfaces.
* Create a BWIPJS instance and make a barcode.

Loading `freetype.js` and `bwip.js` into your JavaScript environment tends to be very platform specific.  If you are working with a CommonJS architecture, then you will likely need to implement a runtime JavaScript loader.  The node-bwipjs file gives an example implementation for node-js: 

```javascript
var fs = require('fs');
var vm = require('vm');

function load(path) {
    var text = fs.readFileSync(path);
    if (!text)
        throw new Error(path + ": could not read file");
    vm.runInThisContext(text, path);
}

// These calls only work if the modules load synchronously.
load('freetype.js');
load('bwip.js');
```

After the two modules are loaded and the BWIPJS global is accessible, you must implement two interfaces:
 * Load and execute bwip-js files on demand.  Many of the encoders rely on additional/secondary encoders.  These dependencies are implemented in the PostScript cross-compile as on-demand loading.  Your host implementation must be able to load and execute bwip-js files after the primary encoder has been invoked.
 * Provide a bitmap graphics interface.  One gotcha here is that you won't know the size of the bitmap until after the encoders have run.  To make it more interesting, you will routinely see negative values for the `x,y` coordinates.  Therefore, creating a barcode image is always a two step process.  First run the encoder/renderer and record each pixel's coordinates, along with the min/max values for x and y.  Once the encoder has returned, you can determine the bitmap dimensions and render the final image.

Also to keep in mind is that typical graphics environments (HTML canvas, PHP GD, etc.) set their origin (0,0) to the upper left-hand corner.  PostScript uses a page-up orientation and the origin is in the lower left-hand corner.  Since bwip-js implements a direct mapping to the PostScript graphics primitives, the coordinates your bitmap implementation will see have an origin set in the lower left-hand corner.  See the files lib/canvas.js and node-bwipjs for bitmap implementations that convert between the two origin conventions.

Below are the function and object prototypes for the interfaces to implement:

```javascript
// Demand-load the BWIPP encoders and renderers.
// This function is installed as a "class static" method on the
// bwip-js constructor.
BWIPJS.load = function(path) {
	// path is relative to the bwip-js root directory.  For example, a call
	// to load the one-dimensional barcode renderer will pass the path:
	//		"bwipp/renlinear.js"
	//
	// This implementation must translate the relative path to whatever is
	// required of the server framework and directory structure.
	// bwip-js tracks what has already been loaded (regardless of whether a
	// module was preloaded or demand-loaded).
	// So this interface will only be called when needed.
	//
	// Based on the server's architecture, you are free to load the module
	// either synchronously or asynchronously.  Each module has an embedded 
	// callback that notifies BWIPJS that it has loaded.
	...
}

// The bitmap interface.  bwip-js will only call the color() and set() 
// methods during encoding/drawing.  A third method is needed by the 
// host code to render the actual barcode image.
// Note that you can name this object class anything you like.  The
// host code is the only one that references it by name.
function Bitmap() {
	this.color = function(r, g, b) {
		// r,g,b will be integer values between 0 and 255
		// Save as the current color
		...
	};
	this.set = function(x, y, a) {
		// x,y will be floating point values.  Convert to int.
		x = Math.floor(x);
		y = Math.floor(y);
		
		// Save the coordinates, the alpha value, and current color
		...
		// Track the min/max values of x and y
		...
	};
	this.render = function( ... ) {
		// This is your code to render the final bitmap image.
		// Width of the bitmap is:  max_x - min_x + 1
		// Height of the bitmap is: max_y - min_y + 1
		//
		// You will need to offset the pixel coordinates by min_x and min_y.
		// And invert the y-axis if the graphics format you are using has
		// its origin in the upper-left corner.
		...
	};
}
```

Once the host framework is implemented, it's time to invoke an encoder.  You must remember that this is a PostScript emulation, so the calling conventions have a PostScript feel to them.  

BWIPP (the PostScript library) uses a two-parameter calling convention.  The first parameter is always the barcode text to be encoded.  The second parameter is the options that govern how the barcode is generated.  This second parameter can be either a string or a PostScript dictionary object.  It is recommended to use a dictionary object for the options, as they provide more flexibility. 

For example, one of the standard options available for linear/one-dimensional barcodes is the `alttext` option.  This option allows overriding what is printed as the human readable component of the barcode.  If the options are specified as a string, no spaces can occur in the `alttext` value.  This restriction does not exist if a dictionary object is used.

To invoke a barcode encoder/renderer, you do the following:

 * Install the host bitmap interface.
 * Create the dictionary object for the options parameter.
 * Add the necessary options to the object.
 * Push the barcode text onto the PostScript operand stack.
 * Push the options object onto the PostScript operand stack.
 * Invoke the encoder.  bwip-js will automatically demand load any required modules.
 
The following example calls the `code128` encoder using the `parsefnc` option to specify the FNC1 character as part of the barcode text, and provides an alternate, human readable text.

```javascript
// Select monochrome or anti-aliased fonts.  You can change the selection
// between calls.
// 0 == anti-aliased.  1 == monochrome.  Default is 0.
BWIPJS.ft_monochrome(0);	// anti-aliased

// Create a barcode writer instance
var bw = new BWIPJS;

// Create the bitmap interface and pass to the emulator
bw.bitmap(new Bitmap);

// Set the scaling factor
bw.scale(2, 2);

// Create a dictionary object and set the options
var opts = {};
opts.parsefnc    = bw.value(true);
opts.includetext = bw.value(true);
opts.alttext     = bw.value("(00)1234567890");

// Push the barcode text and options onto the operand stack
bw.push("^FNC1001234567890");
bw.push(opts);

// Invoke the encoder and render the barcode
bw.call('code128', function (e) {
	if (e) {
		// handle error
	} else {
		bw.bitmap().render();
	}
});
```

## Supported Barcode Types

&#x2022; AusPost 4 State Customer Code &#x2022; Aztec Code &#x2022; Aztec Runes &#x2022; BC412 &#x2022; Channel Code &#x2022; Codabar &#x2022; Codablock F &#x2022; Code 11 &#x2022; Code 128 &#x2022; Code 16K &#x2022; Code 25 &#x2022; Code 39 &#x2022; Code 39 Extended &#x2022; Code 49 &#x2022; Code 93 &#x2022; Code 93 Extended &#x2022; Code One &#x2022; Compact Aztec Code &#x2022; Compact PDF417 &#x2022; COOP 2 of 5 &#x2022; Custom 1D symbology &#x2022; Custom 4 state symbology &#x2022; Data Matrix &#x2022; Datalogic 2 of 5 &#x2022; Deutsche Post Identcode &#x2022; Deutsche Post Leitcode &#x2022; EAN-13 &#x2022; EAN-13 Composite &#x2022; EAN-2 (2 digit addon) &#x2022; EAN-5 (5 digit addon) &#x2022; EAN-8 &#x2022; EAN-8 Composite &#x2022; Flattermarken &#x2022; GS1 Composite 2D Component &#x2022; GS1 Data Matrix &#x2022; GS1 DataBar Expanded &#x2022; GS1 DataBar Expanded Composite &#x2022; GS1 DataBar Expanded Stacked &#x2022; GS1 DataBar Expanded Stacked Composite &#x2022; GS1 DataBar Limited &#x2022; GS1 DataBar Limited Composite &#x2022; GS1 DataBar Omnidirectional &#x2022; GS1 DataBar Omnidirectional Composite &#x2022; GS1 DataBar Stacked &#x2022; GS1 DataBar Stacked Composite &#x2022; GS1 DataBar Stacked Omnidirectional &#x2022; GS1 DataBar Stacked Omnidirectional Composite &#x2022; GS1 DataBar Truncated &#x2022; GS1 DataBar Truncated Composite &#x2022; GS1 QR Code &#x2022; GS1-128 &#x2022; GS1-128 Composite &#x2022; GS1-14 &#x2022; HIBC Codablock F &#x2022; HIBC Code 128 &#x2022; HIBC Code 39 &#x2022; HIBC Data Matrix &#x2022; HIBC MicroPDF417 &#x2022; HIBC PDF417 &#x2022; HIBC QR Code &#x2022; IATA 2 of 5 &#x2022; Industrial 2 of 5 &#x2022; Interleaved 2 of 5 (ITF) &#x2022; ISBN &#x2022; ISMN &#x2022; ISSN &#x2022; Italian Pharmacode &#x2022; ITF-14 &#x2022; Japan Post 4 State Customer Code &#x2022; Matrix 2 of 5 &#x2022; MaxiCode &#x2022; Micro QR Code &#x2022; MicroPDF417 &#x2022; Miscellaneous symbols &#x2022; MSI Modified Plessey &#x2022; PDF417 &#x2022; Pharmaceutical Binary Code &#x2022; Pharmazentralnummer (PZN) &#x2022; Plessey UK &#x2022; PosiCode &#x2022; QR Code &#x2022; Royal Dutch TPG Post KIX &#x2022; Royal Mail 4 State Customer Code &#x2022; SSCC-18 &#x2022; Telepen &#x2022; Telepen Numeric &#x2022; Two-track Pharmacode &#x2022; UPC-A &#x2022; UPC-A Composite &#x2022; UPC-E &#x2022; UPC-E Composite &#x2022; USPS Intelligent Mail &#x2022; USPS PLANET &#x2022; USPS POSTNET &#x2022;


## Node.js

> As of March 2015, the node-bwipjs port is not being actively maintained.
> I am not a node developer and do not have the resources to upgrade and
> test the code.
> 
> If you would like to volunteer to work on the node module,
> please let me know.


## Emulator Notes

bwip-js does not provide a general purpose PostScript emulation layer. The emulation
supports only the PostScript operators used by BWIPP. The emulation is implemented
as a cross-compiler that converts all non-graphics code to equivalent JavaScript,
and converts all graphics operations to calls into a platform-neutral graphics
context. It is up to the host environment to implement the bitmap interface that
is exposed by the graphics context. An example implementation is provided in the
demo. 

The emulation uses, as much as possible, native JavaScript data types:
 * Boolean values true and false
 * Null value
 * Numeric values (integer and float)
 * Objects (PostScript dictionary object)
 * Function objects

You will note that two of the more common JavaScript data types, strings and arrays,
are not on the list.  PostScript implements live-view semantics with strings and
arrays, similar to JavaScript's Typed Arrays and Views.  For that matter, PostScript
strings can be implemented using Uint8Arrays but PostScript arrays have no similar
JavaScript equivalent.  We must wait for ECMAScript6's computed property name getters
and setters to land in the primary browsers before arrays can be handled cleanly.

