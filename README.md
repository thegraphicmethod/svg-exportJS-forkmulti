# svg-exportJS-fork [![Codacy Badge](https://api.codacy.com/project/badge/Grade/a2677830f9d2432d8061a8151e03fd23)](https://app.codacy.com/gh/sharonchoong/svg-exportJS?utm_source=github.com&utm_medium=referral&utm_content=sharonchoong/svg-exportJS&utm_campaign=Badge_Grade)

This is a fork of: https://github.com/sharonchoong/svg-exportJS
It started as a minor modification to the original library to fix a bug, but has since diverged.

An easy-to-use client-side Javascript library to export SVG graphics from web pages and download them as an SVG file, PDF, or raster image (JPEG, PNG) format. Written in plain vanilla javascript. Originally created to export D3.js charts.

Demo: https://thegraphicmethod.github.io/svg-exportJS-forkmulti/examples/browser/index.html

## Features

- Exporting SVG DOM Element objects or serialized SVG string to SVG file, PNG, JPEG, PDF
- Setting custom size for exported image or graphic
- High resolution raster image, using `scale`
- Including external CSS styles in SVG
- Filtering out parts of the SVG by CSS selector
- Exporting text in custom embedded fonts
- Handling transparent background for JPEG format conversion
- Exporting SVGs that are hidden on the DOM (`display: none`, SVGs in hidden modals, dropdowns or tabs, etc.) 
- Exporting SVGs containing images (`<image>` tag)

## Installation

### Option 1: NPM (Recommended)
```bash
npm install svg-export-forkmulti
```

For PDF export functionality:
```bash
npm install pdfkit svg-to-pdfkit blob-stream
```

For PNG/JPEG export functionality:
```bash
npm install canvg
```

Using the module version of the library you can also use the text-to-path feature. Read below for more information: [Text to Path Conversion](#text-to-path-conversion)

### Option 2: Script Tags
```html
<!-- Core library -->
<script src="path/to/svg-export.umd.js"></script>



<!-- Optional: For PNG/JPEG export -->
<script src="https://cdn.skypack.dev/canvg@^4.0.0"></script>

<!-- Optional: For PDF export -->
<script src="https://cdn.jsdelivr.net/npm/pdfkit@0.13.0/js/pdfkit.standalone.js"></script>
<script src="https://cdn.jsdelivr.net/npm/blob-stream-browserify@0.1.3/index.js"></script>
<script src="https://cdn.jsdelivr.net/npm/svg-to-pdfkit@0.1.8/source.js"></script>
```
Please note that the CDNs above may not be the most up-to-date. The latest source code can be found directly from the github projects, also linked above.
## Usage
Depending on your needs, you can use the library in different ways, importing or not importing dependencies.
### ES Modules

Basic SVG export (no dependencies required):
```javascript
import SvgExport from "svg-export-forkmulti";

const exporter = new SvgExport();
exporter.downloadSvg(mySvg, 'chart');
```

With PNG/JPEG export:
You need to import the Canvg library and the presets.
```javascript
import SvgExport from "svg-export-forkmulti";
import { Canvg, presets } from 'canvg';

const exporter = new SvgExport({
    canvg: Canvg,
    presets: presets
});

exporter.downloadPng(mySvg, 'chart');
```

With PDF export:
You need to import the PDFKit library, the SVG-to-PDFKit library and the blob-stream library.
```javascript
import SvgExport from "svg-export-forkmulti";
import PDFDocument from 'pdfkit';
import SVGtoPDF from 'svg-to-pdfkit';
import blobStream from 'blob-stream';

const exporter = new SvgExport({
    pdfkit: PDFDocument,
    svgToPdf: SVGtoPDF,
    blobStream: blobStream
});

exporter.downloadPdf(mySvg, 'chart');
```

With all features:
You need to import the Canvg library, the presets, the PDFKit library, the SVG-to-PDFKit library and the blob-stream library.
```javascript
import SvgExport from "svg-export-forkmulti";
import { Canvg, presets } from 'canvg';
import PDFDocument from 'pdfkit';
import SVGtoPDF from 'svg-to-pdfkit';
import blobStream from 'blob-stream';

const exporter = new SvgExport({
    canvg: Canvg,
    presets: presets,
    pdfkit: PDFDocument,
    svgToPdf: SVGtoPDF,
    blobStream: blobStream
});
```

### Browser Script Tags
In case you want to use the library in a browser environment, with no build step, you can use the script tags version of the library. (check the examples/browser folder for more details)

```javascript
// Basic SVG export
const exporter = new SvgExport();

// With PNG/JPEG export
const exporter = new SvgExport({
    canvg: Canvg,
    presets: presets
});

// With PDF export
const exporter = new SvgExport({
    pdfkit: PDFDocument,
    svgToPdf: SVGtoPDF,
    blobStream: blobStream
});
```

### Example Usage

Given an SVG element:
```html
<svg id="mysvg">...</svg>
```

You can export it like this:
```javascript
// Export as SVG
exporter.downloadSvg(
    document.getElementById("mysvg"), // SVG DOM Element or SVG string
    "chart-name",                     // Output filename
    { width: 200, height: 200 }       // Options (optional)
);

// Export as PNG
exporter.downloadPng(document.getElementById("mysvg"), "chart-name");

// Export as JPEG
exporter.downloadJpeg(document.getElementById("mysvg"), "chart-name");

// Export as PDF
exporter.downloadPdf(document.getElementById("mysvg"), "chart-name");
```

## Options

- **originalWidth** (number) : _declares the original width of the SVG on the DOM. If not specified, the width of the SVG will be computed using getBoundingClientRect or getBBox, but this may not be totally accurate especially for SVGs that have been transformed_ 
- **originalHeight** (number) : _declares the original height of the SVG on the DOM. If not specified, the height of the SVG will be computed using getBoundingClientRect or getBBox, but this may not be totally accurate especially for SVGs that have been transformed_ 
- **scale** (number) : _a multiple by which the SVG can be increased or decreased in size. The default scale is `1`_
- **useCSS** (bool): _if SVG styles are specified in stylesheet externally rather than inline, setting `true` will add references to such styles from the styles computed by the browser. If useCSS is `false`, `currentColor` will be changed to `black`. This setting only applies if the SVG is passed as a DOM Element object, not as a string. Set this to `false` whenever possible to optimize performance. When set to `true`, all elements in the SVG are iterated to obtain their computed styles, which can be costly for large SVGs (please read **Optimizing for large SVGs** below for more detail). Default is `true`_
- **excludeByCSSSelector** (string): _e.g. `[stroke='red'], [stroke='green'], [display='none'], .text-muted`. Elements matching the specified [CSS selector](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors) will not be included in the generated file. This can be used to remove unwanted/unsupported elements of the SVG from the exported file, or to optimize performance for large SVGs. Please read **Optimizing for large SVGs** and **Not Supported** below for more detail._
- **transparentBackgroundReplace** (string): _the color to be used to replace a transparent background in JPEG format export. Default is `white`_
- **allowCrossOriginImages** (bool): _If the SVG contains images, this option permits the use of images from foreign origins. Defaults to `false`. Please read **Images within SVG** below for more detail._
- **onDone** (function): _an optional callback function that will be called after the file export has completed._
- **pdfOptions**
  - **pageLayout** (object): _e.g. `{ margin: 50, layout: "landscape" }`. This is provided to PDFKit's `addPage`. When the options **width** and **height** are not specified, a minimum size of 300x300 is used for the PDF page size; otherwise the page size wraps around the SVG size. Please see the [PDFKit documentation](https://pdfkit.org/docs/getting_started.html#adding_pages) for more info_
  - **addTitleToPage** (bool): _Default is `true`_
  - **chartCaption** (string) _caption to appear at the bottom of the chart in the PDF. Default is no caption_
  - **pdfTextFontFamily** (string): _Font family of title and caption (if applicable) in PDF. See here for a [list of available fonts](http://pdfkit.org/docs/text.html#fonts). Default is `Helvetica`_
  - **pdfTitleFontSize** (number): _Default is `20`_
  - **pdfCaptionFontSize** (number): _Default is `14`_
  - **customFonts** (array of objects): _Optional argument for custom fonts. e.g. `[{ fontName: 'FakeFont', url: 'fonts/FakeFont.ttf'}]`. Each object must have two properties: `fontName` for the font name that appears in the CSS/SVG, and `url` for the URL of the custom font file to be used in the PDF. A third property `styleName` specifying the style name to be used can be specified for multi-collection font files (.ttc and .dfont files)_
  - **convertTextToPath** (bool): _when true, converts text elements to path elements before export. Default is `false`_
  - **svgTextToPathSettings** (object): _configuration options for text-to-path conversion_

### Custom fonts

Regarding embedded custom fonts used in the SVG element (using @font-face for example), please note that for SVG export, custom fonts only show correctly if the system opening the SVG file has the font installed. If this is not possible, please consider using one of the other file formats available. (Read more about custom fonts in the [Text to Path Conversion](#text-to-path-conversion) section)

### Images within SVG
This library supports exporting SVGs that contain images in an `<image>` tag. If you need to export such SVGs to raster images or PDFs, please make sure that you have the latest version of Canvg and SVG-to-PDFKit. If the images' `href` are external, the `allowCrossOriginImages` option must be set to `true`, and the image servers must be configured with the ['Access-Control-Allow-Origin'](https://developer.mozilla.org/en-US/docs/Web/HTML/CORS_enabled_image) CORS policy. 

### Optimizing for large SVGs
- Set the `useCSS` option to `false` whenever possible to optimize performance. When set to `true`, all elements in the SVG are iterated to obtain their computed styles, which can be costly for large SVGs.
- If you have no choice but to set `useCSS` to `true` for a large SVG, but it is causing slow performance or a frozen browser, you can also try filtering out unneeded elements within the SVG, using the `excludeByCSSSelector` setting. For example, you could exclude all elements in the SVG that are styled as `display: none`, exclude elements that have the attribute `fill=transparent`, or exclude unneeded elements that have a specific class name.

### Colors

Colors tested to work on all exported formats include CSS color names, HEX, RGB, RGBA and HSL.

### SVG graphics in Office documents

Need to add SVG graphics to Office Word, Excel or Powerpoint presentations? [SVG files can be inserted as a picture](https://support.microsoft.com/en-us/office/edit-svg-images-in-microsoft-office-365-69f29d39-194a-4072-8c35-dbe5e7ea528c) for non-pixelated graphics in Office 2016 or later, including Office 365.

### Text to Path Conversion (Optional)

To convert text elements to path elements (useful for ensuring text appears correctly regardless of font availability), you can use the text-to-path feature, thanks to [svg-text-to-path](https://github.com/paulzi/svg-text-to-path).

Note: Using the "text to path" feature actually modifies the original SVG in the DOM.

#### Installation

```bash
npm install svg-text-to-path
```

#### Usage with NPM/ES Modules

```javascript
import SvgExport from "svg-export-forkmulti";
import Session from 'svg-text-to-path';

const exporter = new SvgExport({
    textToPath: Session
});

exporter.downloadSvg(mySvg, 'chart', {
    convertTextToPath: true,
    svgTextToPathSettings: {
        fonts: {
            // Font configurations https://github.com/paulzi/svg-text-to-path/blob/master/documentation.md
            'MyFontFamily': [{ 
                source: '/path/to/font.ttf'
            }]
        }
    }
});


#### Text to Path Options

- **convertTextToPath** (bool): _when true, converts text elements to path elements before export. Default is `false`_
- **svgTextToPathSettings** (object): _configuration options for text-to-path conversion_
  [text-to-path documentation](https://github.com/paulzi/svg-text-to-path/blob/master/documentation.md)
    ```

This feature is particularly useful when:
- You need to ensure text appears exactly the same across different systems
- Your SVG uses custom fonts that might not be available on the target system
- You want to convert text to vector paths for better compatibility

## Not Supported
Since `foreignObject` does not contain SVG, it is not supported.

## Contributing

Contributions are very welcome! Feel free to flag issues or pull requests.

## License

Licensed under MIT. See `LICENSE` for more information.

## Contact

Original author:
Sharon Choong - [https://sharonchoong.github.io/](https://sharonchoong.github.io)
 
Fork author Sergio Galán - [https://www.graphicmethod.studio/](https://www.graphicmethod.studio/)    
