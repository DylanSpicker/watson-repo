---
layout: post
title:  "Visual Recognition - Template"
img: "imgs/svg/VisualRecognition.svg"
description: "The visual recognition service allows for the content of images to be classified, based either on a pre-trained model, or a custom classifier. An easy-to-use training interface provides a great way to build a custom image classifier using Watson."
bm_url: "https://console.bluemix.net/catalog/services/visual-recognition"
---
# Visual Recognition

The visual recognition service offers an interesting domain for Watson, which is not captured by the other services typically. Because of this, the sample code to ensure a functioning application has more moving parts than the other templated examples provided. IBM provides an example application, written in NodeJS ([View on Github](https://github.com/watson-developer-cloud/visual-recognition-nodejs)), but does not provide a readily available Python demo.

Templated code provided in Python would likely do little to educate on the service; instead, the following resources could be brought together to generate a usable application built on the Visual Recognition service, in Python.

## External Tutorials
[File Uploads in Flask:](http://flask.pocoo.org/docs/0.12/patterns/fileuploads/) This tutorial page will walk you through how to allow users to upload files in Flask, allowing for an image to be taken.

[Image Uploads using Flask Uploads:](http://www.patricksoftwareblog.com/tag/flask-uploads/) This tutorial leverages a Flask extensions (Flask Uploads) to demonstrate how you can easily manage file uploads (specifically images), in a more scalable manner.

[Accessing a Webcam with HTML5: ](https://www.kirupa.com/html5/accessing_your_webcam_in_html5.htm) This tutorial will show you how to access a webcam feed from an HTML window, in the browser. In this manner you could classify images directly from the webcam.

[Python Visual Recognition API: ](https://www.youtube.com/watch?v=GFjYqzdWIJk) This tutorial will simply walk you through the calling of the Visual Recognition API. If you are comfortable with the other Bluemix APIs, it is likely unnecessary, but can be helpful in addition to the documentation.