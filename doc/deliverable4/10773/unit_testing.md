# Unit Testing

The unit test is done using testing framework jasmine, which is located in `test/unit/jasmine-boot.js` and modified for loading PDF.js libraries.

## Instructions

### Step 1

Clone the repository with the changes

### Step 2

Build according to the instructions on the README document

### Step 3

Start the server to use the viewer locally by using `gulp server`

### Step 4

Run the unit tests using the following link:
```
http://localhost:8888/test/unit/unit_test.html
```
The result of all unit tests will be displyed on the page, with green dot indicating passing and red cross mark indicating failures, as shown in the screenshot.

![Unit test](./img/unit_test.png)

## Mock Object

When testing the modified component related to [issue #10773](https://github.com/mozilla/pdf.js/issues/10773), mock class, `BaseViewerMock`, was added in [`test/unit/test_utils.js`](https://github.com/CSCD01/pdf.js-team22/blob/47a40309ccea149f6441dd504048aa0057872126/test/unit/test_utils.js#L178-L222) in order to replicate the behaviour of [`BaseViewer`](https://github.com/CSCD01/pdf.js-team22/blob/4fe92605b75d7e0952738b7f1575d78145b69aeb/web/base_viewer.js#L135).

```
  class BaseViewerMock {
    constructor(){
      this.used = 0;
      this.x = 0;
      this.y = 0;
      this.desName = "";
    }
    scrollPageIntoView({
      pageNumber,
      destArray = null,
      allowNegativeOffset = false,
      ignoreDestinationZoom = false,
    }){
      this.used = 1;
      this.desName = destArray[1].name;
      var x = destArray[2];
      this.x = x !== null ? x : 0;
    } 
    
    reset(){
      this.used = 0;
      this.x = 0;
      this.desName = "";
    }

    getUsed(){
      return this.used;
    }
    
    getX(){
      return this.x;
    }

    getName(){
      return this.desName;
    }
  }
```

## Test Cases

[1](https://github.com/CSCD01/pdf.js-team22/blob/47a40309ccea149f6441dd504048aa0057872126/test/unit/10773_unit_test.js#L26-L52). Calling mock class to add different fit parameters

[2](https://github.com/CSCD01/pdf.js-team22/blob/47a40309ccea149f6441dd504048aa0057872126/test/unit/10773_unit_test.js#L54-L78). If the correct fit parameter is set

[3](https://github.com/CSCD01/pdf.js-team22/blob/47a40309ccea149f6441dd504048aa0057872126/test/unit/10773_unit_test.js#L80-L98). If the correct coordinates are set with the given fit parameter