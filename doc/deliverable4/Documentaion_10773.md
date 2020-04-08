# Feature

[10773](https://github.com/CSCD01/team_22-project/blob/master/doc/deliverable3/10773.md)

We switched to this feature after [#6347](./Documentaion_6347.md). We also tried to fix a related issue related to this feature [#2843](https://github.com/mozilla/pdf.js/issues/2843) during implementation.

## Description

### 10773
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

### 2843
According to the open parameter specification, [link](http://www.adobe.com/content/dam/Adobe/en/devnet/acrobat/pdfs/pdf_open_parameters_v9.pdf#page=6), the zoom parameter should be implemented as: `zoom=scale,left,top`, with the interpretation:

> Scroll values _left_ and _top_ are in a coordinate system where 0,0 which represents the top left corner of the visible page, regardless of document rotation.

This is not the case in pdf.js, which is a problem if you try opening the following:
http://mozilla.github.io/pdf.js/web/viewer.html#page=2&zoom=auto,0,0

* Expected result: The document should open at the top of page 2, as it does with Adobe Reader.
* What actually happens: The document is scrolled down to the top of page 3.
 
The problem seems to be that in pdf.js, the origin in the coordinate system (the one that is exposed to the user) is placed at the bottom left corner. From what I've gathered, pdf.js internally uses a coordinate system placed at the top left corner of the page, so I don't know why it's not working.
I'm not so familiar with this part of the codebase, but could be the issue here:
https://github.com/mozilla/pdf.js/blob/master/src/util.js#L390


## Design in Code

![UML](https://github.com/CSCD01/team_22-project/blob/master/doc/deliverable4/img/10773_UML_2.png)

## Implementation

The related [Pull Request](https://github.com/mozilla/pdf.js/pull/11786) is created.

### 10773

In [web/pdf_link_service.js](https://github.com/mozilla/pdf.js/blob/master/web/pdf_link_service.js)

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
And here, we also delete the duplicate part in zoom, based on project advisors' suggestion. Then previous FitV, FitH etc. passing through `zoom` is now going through `view` in url.
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
    } 
```

Regardless of #2843, the feature for #10773 is now fixed. It meets the PDF standard and the user can set up view to change default fit pattern on web.

### 2843
After tons of testing, we find out the issue is in `base_viewer.js` it's [converting](https://github.com/mozilla/pdf.js/blob/4fe92605b75d7e0952738b7f1575d78145b69aeb/web/base_viewer.js#L885-L890) input points to a from-top-left point.
```
    const boundingRect = [
      pageView.viewport.convertToViewportPoint(x, y),
      pageView.viewport.convertToViewportPoint(x + width, y + height),
    ];
    let left = Math.min(boundingRect[0][0], boundingRect[1][0]);
    let top = Math.min(boundingRect[0][1], boundingRect[1][1]);
```
Thus, the input value we passed from `pdf_link_service.js` is supposed to be a from-botton-left point. However, from the user point, their input is actually a from-top-left value. Due to that, there is an extra conversion and massed up the position value.

As discussed in [comment](https://github.com/mozilla/pdf.js/issues/10773#issuecomment-610183217), we initially decided to apply and addtional conversion in `base_viewer.js` to make all the position value from-botton-left. But this was rejected by the reviewer, who said this way would affact other part of project so it's not safe and elegent enough. We think that's an reasonable rejection and we accept it.

After that, the other solution we can think about is to have another wrapper out side [base_viewer.js/scrollPageIntoView](https://github.com/mozilla/pdf.js/blob/4fe92605b75d7e0952738b7f1575d78145b69aeb/web/base_viewer.js#L774) to be use by `zoom` and `view` only, but that's still not elegent enough.

We are still keeping track with the reviewer to get suggestions on what we can do to make this fixed.

## Acceptance Testing
We expect to see the default view of pdf changed base on the parameter pass in. This can be tested by running web part locally with gulp server. To verify, for example, by navigating through the url below from browser:
```
http://localhost:8888/web/viewer.html?file=%2Ftest%2Fpdfs%2Ftracemonkey.pdf
```
![before](https://github.com/CSCD01/team_22-project/blob/master/doc/deliverable3/img/10773_1.PNG)
Currently we are going to see above with default setting, but after implementation, we can show below with the view parameter. For example, when fitting horizontally,
```
http://localhost:8888/web/viewer.html?file=%2Ftest%2Fpdfs%2Ftracemonkey.pdf#page=1&view=FitH
```
![after](https://github.com/CSCD01/team_22-project/blob/master/doc/deliverable3/img/10773_2.PNG)
We would see above the same pdf is fitted by width by default.

## User Guide

After the implementation is done, we will have options in url such as append `#page=1&view=FitV,100` to fit the page vertically with 100px from top.

