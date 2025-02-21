[<img width="250" alt="ImageKit.io" src="https://raw.githubusercontent.com/imagekit-developer/imagekit-javascript/master/assets/imagekit-light-logo.svg"/>](https://imagekit.io)

# ImageKit.io Javascript SDK

![gzip size](https://img.badgesize.io/https://unpkg.com/imagekit-javascript/dist/imagekit.min.js?compression=gzip&label=gzip)
![brotli size](https://img.badgesize.io/https://unpkg.com/imagekit-javascript/dist/imagekit.min.js?compression=brotli&label=brotli)
![Node CI](https://github.com/imagekit-developer/imagekit-javascript/workflows/Node%20CI/badge.svg)
[![npm version](https://img.shields.io/npm/v/imagekit-javascript)](https://www.npmjs.com/package/imagekit-javascript) 
[![codecov](https://codecov.io/gh/imagekit-developer/imagekit-javascript/branch/master/graph/badge.svg)](https://codecov.io/gh/imagekit-developer/imagekit-javascript)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Twitter Follow](https://img.shields.io/twitter/follow/imagekitio?label=Follow&style=social)](https://twitter.com/ImagekitIo)

Javascript SDK for [ImageKit](https://imagekit.io/) provides URL generation for image & video resizing and provides an interface for file upload. This SDK is lightweight and has no dependency. You can also use this as an ES module.

ImageKit is complete media storage, optimization, and transformation solution that comes with an [image and video CDN](https://imagekit.io/features/imagekit-infrastructure). It can be integrated with your existing infrastructure - storage like AWS S3, web servers, your CDN, and custom domain names, allowing you to deliver optimized images in minutes with minimal code changes.

## Installation

### Using npm
Install `imagekit-javascript`
```
npm install imagekit-javascript --save
#or
yarn add imagekit-javascript
```

Now import ImageKit
```js
import ImageKit from "imagekit-javascript"

// or
const ImageKit = require("imagekit-javascript")
```

### Using CDN
You can download a specific version of this SDK from a global CDN.
```
https://unpkg.com/imagekit-javascript@1.3.0/dist/imagekit.min.js
```

For the latest version, remove the version number i.e.
```
https://unpkg.com/imagekit-javascript/dist/imagekit.min.js
```

Now load it using a `<script>` tag.

```
<script type="text/javascript" src="https://unpkg.com/imagekit-javascript/dist/imagekit.min.js"></script>
```

## Initialization

`urlEndpoint` is required to use the SDK. You can get URL-endpoint from your ImageKit dashboard - https://imagekit.io/dashboard#url-endpoints

```
var imagekit = new ImageKit({
    urlEndpoint: "https://ik.imagekit.io/your_imagekit_id"
});    
```

`publicKey` and `authenticationEndpoint` parameters are required if you want to use the SDK for client-side file upload. You can get these parameters from the developer section in your ImageKit dashboard - https://imagekit.io/dashboard#developers
```
var imagekit = new ImageKit({
    publicKey: "your_public_api_key",
    urlEndpoint: "https://ik.imagekit.io/your_imagekit_id",
    authenticationEndpoint: "http://www.yourserver.com/auth",
});    
```

*Note: Do not include your Private Key in any client-side code, including this SDK or its initialization. If you pass the `privateKey` parameter while initializing this SDK, it throws an error*

## Demo Application

The fastest way to get started is by running the demo application in [samples/sample-app](https://github.com/imagekit-developer/imagekit-javascript/tree/master/samples/sample-app) folder. Follow these steps to run the application locally:

```
git clone https://github.com/imagekit-developer/imagekit-javascript.git

cd imagekit-javascript
```

Create a file `.env` using `sample.env` in the directory `samples/sample-app` and fill in your `PRIVATE_KEY`, `PUBLIC_KEY` and `URL_ENDPOINT` from your [imageKit dashboard](https://imagekit.io/dashboard#developers). `SERVER_PORT` must also be included as per the `sample.env` file.

Now start the sample application by running:

```
// Run it from project root
yarn startSampleApp
```

## Usage
You can use this SDK for URL generation and client-side file uploads.

### URL Generation

**1. Using image path and image hostname or endpoint**

This method allows you to create an URL to access a file using the relative file path and the ImageKit URL endpoint (`urlEndpoint`). The file can be an image, video, or any other static file supported by ImageKit.

```
var imageURL = imagekit.url({
    path: "/default-image.jpg",
    urlEndpoint: "https://ik.imagekit.io/your_imagekit_id/endpoint/",
    transformation: [{
        "height": "300",
        "width": "400"
    }]
});
```

This results in a URL like

```
https://ik.imagekit.io/your_imagekit_id/endpoint/tr:h-300,w-400/default-image.jpg
```

**2. Using full image URL**

This method allows you to add transformation parameters to an absolute URL. For example, if you have configured a custom CNAME and have absolute asset URLs in your database or CMS, you will often need this.


```
var imageURL = imagekit.url({
    src: "https://ik.imagekit.io/your_imagekit_id/endpoint/default-image.jpg",
    transformation: [{
        "height": "300",
        "width": "400"
    }]
});
```

This results in a URL like

```
https://ik.imagekit.io/your_imagekit_id/endpoint/default-image.jpg?tr=h-300%2Cw-400
```


The `.url()` method accepts the following parameters

| Option           | Description                    |
| :----------------| :----------------------------- |
| urlEndpoint      | Optional. The base URL to be appended before the path of the image. If not specified, the URL Endpoint specified at the time of SDK initialization is used. For example, https://ik.imagekit.io/your_imagekit_id/endpoint/ |
| path             | Conditional. This is the path at which the image exists. For example, `/path/to/image.jpg`. Either the `path` or `src` parameter needs to be specified for URL generation. |
| src              | Conditional. This is the complete URL of an image already mapped to ImageKit. For example, `https://ik.imagekit.io/your_imagekit_id/endpoint/path/to/image.jpg`. Either the `path` or `src` parameter needs to be specified for URL generation. |
| transformation   | Optional. An array of objects specifying the transformation to be applied in the URL. The transformation name and the value should be specified as a key-value pair in the object. Different steps of a [chained transformation](https://docs.imagekit.io/features/image-transformations/chained-transformations) can be specified as different objects of the array. The complete list of supported transformations in the SDK and some examples of using them are given later. If you use a transformation name that is not specified in the SDK, it gets applied as it is in the URL. |
| transformationPostion | Optional. The default value is `path`, which places the transformation string as a path parameter in the URL. It can also be specified as `query`, which adds the transformation string as the query parameter `tr` in the URL. If you use the `src` parameter to create the URL, then the transformation string is always added as a query parameter. |
| queryParameters  | Optional. These are the other query parameters that you want to add to the final URL. These can be any query parameters and are not necessarily related to ImageKit. Especially useful if you want to add some versioning parameters to your URLs. |

#### Examples of generating URLs

**1. Chained Transformations as a query parameter**
```
var imageURL = imagekit.url({
    path: "/default-image.jpg",
    urlEndpoint: "https://ik.imagekit.io/your_imagekit_id/endpoint/",
    transformation: [{
        "height": "300",
        "width": "400"
    }, {
        "rotation": 90
    }],
    transformationPosition: "query"
});
```
```
https://ik.imagekit.io/your_imagekit_id/endpoint/default-image.jpg?tr=h-300%2Cw-400%3Art-90
```

**2. Sharpening and contrast transforms and a progressive JPG image**

There are some transforms like [Sharpening](https://docs.imagekit.io/features/image-transformations/image-enhancement-and-color-manipulation) that can be added to the URL with or without any other value. To use such transforms without specifying a value, specify the value as "-" in the transformation object. Otherwise, specify the value that you want to be added to this transformation.

```
var imageURL = imagekit.url({
    src: "https://ik.imagekit.io/your_imagekit_id/endpoint/default-image.jpg",
    transformation: [{
        "format": "jpg",
        "progressive": "true",
        "effectSharpen": "-",
        "effectContrast": "1"
    }]
});
```
```
//Note that because `src` parameter was used, the transformation string gets added as a query parameter `tr`
https://ik.imagekit.io/your_imagekit_id/endpoint/default-image.jpg?tr=f-jpg%2Cpr-true%2Ce-sharpen%2Ce-contrast-1
```

#### List of supported transformations

See the complete list of transformations supported in ImageKit [here](https://docs.imagekit.io/features/image-transformations). The SDK gives a name to each transformation parameter e.g. `height` for `h` and `width` for `w` parameter. It makes your code more readable. If the property does not match any of the following supported options, it is added as it is.

If you want to generate transformations in your application and add them to the URL as it is, use the `raw` parameter.

| Supported Transformation Name | Translates to parameter |
|-------------------------------|-------------------------|
| height | h |
| width | w |
| aspectRatio | ar |
| quality | q |
| crop | c |
| cropMode | cm |
| x | x |
| y | y |
| focus | fo |
| format | f |
| radius | r |
| background | bg |
| border | b |
| rotation | rt |
| blur | bl |
| named | n |
| overlayX | ox |
| overlayY | oy |
| overlayFocus | ofo |
| overlayHeight | oh |
| overlayWidth | ow |
| overlayImage | oi |
| overlayImageTrim | oit |
| overlayImageAspectRatio | oiar |
| overlayImageBackground | oibg |
| overlayImageBorder | oib |
| overlayImageDPR | oidpr |
| overlayImageQuality | oiq |
| overlayImageCropping | oic |
| overlayImageTrim | oit |
| overlayText | ot |
| overlayTextFontSize | ots |
| overlayTextFontFamily | otf |
| overlayTextColor | otc |
| overlayTextTransparency | oa |
| overlayAlpha | oa |
| overlayTextTypography | ott |
| overlayBackground | obg |
| overlayTextEncoded | ote |
| overlayTextWidth | otw |
| overlayTextBackground | otbg |
| overlayTextPadding | otp |
| overlayTextInnerAlignment | otia |
| overlayRadius | or |
| progressive | pr |
| lossless | lo |
| trim | t |
| metadata | md |
| colorProfile | cp |
| defaultImage | di |
| dpr | dpr |
| effectSharpen | e-sharpen |
| effectUSM | e-usm |
| effectContrast | e-contrast |
| effectGray | e-grayscale |
| original | orig |
| raw | The string provided in raw will be added in the URL as it is. |


### File Upload

The SDK provides a simple interface using the `.upload()` method to upload files to the ImageKit Media Library. 

The `upload()` method requires mandatory `file` and the `fileName` parameter. In addition, it accepts all the parameters supported by the [ImageKit Upload API](https://docs.imagekit.io/api-reference/upload-file-api/client-side-file-upload).

Also, ensure that you have specified `authenticationEndpoint` during SDK initialization. The SDK makes an HTTP GET request to this endpoint and expects a JSON response with three fields, i.e. `signature`, `token`, and `expire`. In addition, the SDK adds a query parameter `t` with a random value to ensure that the request URL is unique and the response is not cached in [Safari iOS](https://github.com/imagekit-developer/imagekit-javascript/issues/59). Your backend can ignore this query parameter.

[Learn how to implement authenticationEndpoint](https://docs.imagekit.io/api-reference/upload-file-api/client-side-file-upload#how-to-implement-authenticationendpoint-endpoint) on your server.

You can pass other parameters supported by the ImageKit upload API using the same parameter name as specified in the upload API documentation. For example, to specify tags for a file at the time of upload, use the `tags` parameter as specified in the [documentation here](https://docs.imagekit.io/api-reference/upload-file-api/client-side-file-upload).


#### Sample usage
```html
<form action="#" onsubmit="upload()">
    <input type="file" id="file1" />
    <input type="submit" />
</form>
<script type="text/javascript" src="../dist/imagekit.js"></script>

<script>
    /* SDK initilization
     authenticationEndpoint should be implemented on your server. Learn more here - https://docs.imagekit.io/api-reference/upload-file-api/client-side-file-upload#how-to-implement-authenticationendpoint-endpoint
    */
    var imagekit = new ImageKit({
        publicKey: "your_public_api_key",
        urlEndpoint: "https://ik.imagekit.io/your_imagekit_id",
        authenticationEndpoint: "http://www.yourserver.com/auth"
    });
    
    // Upload function internally uses the ImageKit.io javascript SDK
    function upload(data) {
        var file = document.getElementById("file1");

        // Using Callback Function
        imagekit.upload({
            file: file.files[0],
            fileName: "abc1.jpg",
            tags: ["tag1"],
            extensions: [
                {
                    name: "aws-auto-tagging",
                    minConfidence: 80,
                    maxTags: 10
                }
            ]
        }, function(err, result) {
            console.log(result);
        })

        // Using Promises
        imagekit.upload({
            file: file.files[0],
            fileName: "abc1.jpg",
            tags: ["tag1"],
            extensions: [
                {
                    name: "aws-auto-tagging",
                    minConfidence: 80,
                    maxTags: 10
                }
            ]
        }).then(result => {
            console.log(result);
        }).then(error => {
            console.log(error);
        })
    }
</script>
```

If the upload succeeds, `err` will be `null`, and the `result` will be the same as what is received from ImageKit's servers.
If the upload fails, `err` will be the same as what is received from ImageKit's servers, and the `result` will be null.

## Tracking upload progress using custom XMLHttpRequest
You can use a custom XMLHttpRequest object as the following to bind `progress` or any other events for a customized implementation.

```js
var fileSize = file.files[0].size;
var customXHR = new XMLHttpRequest();
customXHR.upload.addEventListener('progress', function (e) {
    if (e.loaded <= fileSize) {
        var percent = Math.round(e.loaded / fileSize * 100);
        console.log(`Uploaded ${percent}%`);
    } 

    if(e.loaded == e.total){
        console.log("Upload done");
    }
});

imagekit.upload({
    xhr: customXHR,
    file: file.files[0],
    fileName: "abc1.jpg",
    tags: ["tag1"],
    extensions: [
        {
            name: "aws-auto-tagging",
            minConfidence: 80,
            maxTags: 10
        }
    ]
}).then(result => {
    console.log(result);
}).then(error => {
    console.log(error);
})
```

## Access request-id, other response headers, and HTTP status code
You can access `$ResponseMetadata` on success or error object to access the HTTP status code and response headers.

```js
// Success
var response = await imagekit.upload({
    file: file.files[0],
    fileName: "abc1.jpg",
    tags: ["tag1"],
    extensions: [
        {
            name: "aws-auto-tagging",
            minConfidence: 80,
            maxTags: 10
        }
    ]
});
console.log(response.$ResponseMetadata.statusCode); // 200

// { 'content-length': "300", 'content-type': 'application/json', 'x-request-id': 'ee560df4-d44f-455e-a48e-29dfda49aec5'}
console.log(response.$ResponseMetadata.headers);

// Error
try {
    await imagekit.upload({
        file: file.files[0],
        fileName: "abc1.jpg",
        tags: ["tag1"],
        extensions: [
            {
                name: "aws-auto-tagging",
                minConfidence: 80,
                maxTags: 10
            }
        ]
    });
} catch (ex) {
    console.log(response.$ResponseMetadata.statusCode); // 400

    // {'content-type': 'application/json', 'x-request-id': 'ee560df4-d44f-455e-a48e-29dfda49aec5'}
    console.log(response.$ResponseMetadata.headers);
}