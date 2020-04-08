# Feature

[10773](https://github.com/CSCD01/team_22-project/blob/Documentaion_process/doc/deliverable3/10773.md)

## Description

```
view=Fit
view=FitH
view=FitH,top
view=FitV
view=FitV,left
view=FitB
view=FitBH
view=FitBH,top
view=FitBV
view=FitBV,left
```

Set the view of the displayed page, using the keyword values defined in the PDF language specification. Scroll values left and top are floats or integers in a coordinate system where 0,0 represents the top left corner of the visible page, regardless of document rotation.

Steps to get the problem:

1. Check out the repository, run npm install and gulp server, then navigate the viewer to a PDF that isn't automatically zoomed in to fill your viewport width on viewer initialization (http://localhost:8888/web/viewer.html?file=%2Ftest%2Fpdfs%2Ftracemonkey.pdf works for me).
2. Append #page=1&view=FitH to the URL in the address bar, and reload the page.

Currently, the view parameter in PDF URL fragment identifiers is currently unsupported.

What we did is that, add support for "view" parameter for opening PDF files on web following Adobe Acrobat SDk documentation. This parameter would set the view of the displayed page, using the keyword values defined in the PDF language specification. 

## Location in Code

![UML](https://github.com/CSCD01/team_22-project/blob/Documentaion_process/doc/deliverable3/img/10773_UML.png)

## Design of code

First, 

in [web/pdf_link_service.js](https://github.com/mozilla/pdf.js/blob/master/web/pdf_link_service.js)

add code inside of `setHash` function:

```
if ("view" in params) {
        const viewArgs = params.view.split(","); // scale,left/top
        const viewArg = viewArgs[0];

        if (viewArg === "Fit" || viewArg === "FitB") {
          dest = [null, { name: viewArg }];
        } else if (
          viewArg === "FitH" ||
          viewArg === "FitBH" ||
          viewArg === "FitV" ||
          viewArg === "FitBV"
        ) {
          dest = [
            null,
            { name: viewArg },
            viewArgs.length > 1 ? viewArgs[1] | 0 : null,
          ];
        } else {
          console.error(
            `PDFLinkService.setHash: "${viewArg}" is not ` +
              "a valid view value."
          );
        }
      }
```
This aims at build the destination array and make the view of the displayed page available in function, which is similar way as the [zoom](https://github.com/CSCD01/pdf.js-team22/blob/4893b14a522f6aced286d7fd2f4c79dd2807f6f0/web/pdf_link_service.js#L243):

```
if ("zoom" in params) {
        // Build the destination array.
        const zoomArgs = params.zoom.split(","); // scale,left,top
        const zoomArg = zoomArgs[0];
        const zoomArgNumber = parseFloat(zoomArg);

        if (!zoomArg.includes("Fit")) {
          // If the zoomArg is a number, it has to get divided by 100. If it's
          // a string, it should stay as it is.
          dest = [
            null,
            { name: "XYZ" },
            zoomArgs.length > 1 ? zoomArgs[1] | 0 : null,
            zoomArgs.length > 2 ? zoomArgs[2] | 0 : null,
            zoomArgNumber ? zoomArgNumber / 100 : zoomArg,
          ];
        } else {
          if (zoomArg === "Fit" || zoomArg === "FitB") {
            dest = [null, { name: zoomArg }];
          } else if (
            zoomArg === "FitH" ||
            zoomArg === "FitBH" ||
            zoomArg === "FitV" ||
            zoomArg === "FitBV"
          ) {
            dest = [
              null,
              { name: zoomArg },
              zoomArgs.length > 1 ? zoomArgs[1] | 0 : null,
            ];
          } else if (zoomArg === "FitR") {
            if (zoomArgs.length !== 5) {
              console.error(
                'PDFLinkService.setHash: Not enough parameters for "FitR".'
              );
            } else {
              dest = [
                null,
                { name: zoomArg },
                zoomArgs[1] | 0,
                zoomArgs[2] | 0,
                zoomArgs[3] | 0,
                zoomArgs[4] | 0,
              ];
            }
          } else {
            console.error(
              `PDFLinkService.setHash: "${zoomArg}" is not ` +
                "a valid zoom value."
            );
          }
        }
      }
```
And here, we also delete the duplicate part based on project advisors' suggestion:
```
if (zoomArg === "Fit" || zoomArg === "FitB") {
            dest = [null, { name: zoomArg }];
          } else if (
            zoomArg === "FitH" ||
            zoomArg === "FitBH" ||
            zoomArg === "FitV" ||
            zoomArg === "FitBV"
          ) {
            dest = [
              null,
              { name: zoomArg },
              zoomArgs.length > 1 ? zoomArgs[1] | 0 : null,
            ];
          } else if (zoomArg === "FitR") {
```

Second, we need to add script that when the horizontal/vertical scaling differs significantly, also scale even single-char text to improve highlighting in [display/text_layer.js](https://github.com/mozilla/pdf.js/blob/master/src/display/text_layer.js):

```
let shouldScaleText = false;
    if (geom.str.length > 1) {
      shouldScaleText = true;
    } else if (geom.transform[0] !== geom.transform[3]) {
      const absScaleX = Math.abs(geom.transform[0]),
        absScaleY = Math.abs(geom.transform[3]);
      if (
        absScaleX !== absScaleY &&
        Math.max(absScaleX, absScaleY) / Math.min(absScaleX, absScaleY) > 1.5
      ) {
        shouldScaleText = true;
      }
    }
    if (shouldScaleText) {
```

Finally, add the fixed feature into [test/pdfs/.gitignore](https://github.com/mozilla/pdf.js/blob/master/test/pdfs/.gitignore#146) and [test/test_manifest.json](https://github.com/mozilla/pdf.js/blob/master/test/test_manifest.json#L1110)

```
{  "id": "issue11713",
       "file": "pdfs/issue11713.pdf",
       "md5": "bafe5801234feeb95969da106f2ce6d8",
       "rounds": 1,
       "type": "text"
    }
```

## User Guide

After the implementation is done, we will have more options in 

<img src="./img/option_1.png" alt="Option" width="300"/>

like FitH(top), FitV(left), FitB(H)...

