---
layout: post
title:  "Tone Analyzer - Template"
date:   2017-08-23 08:40:39 -0400
categories: jekyll update
---
# Tone Analyzer
The [Tone Analyzer](https://console.bluemix.net/docs/services/tone-analyzer/getting-started.html) is a Watson Service that is used to gauge the emotional, social, and language tones found within a writing sample.

1. Generate Credentials on Bluemix

## The Template
This template will allow you to provide an input field for the user, have them click the submit button, and have the tone scores returned to the user. The template assumes you are using [Flask](http://flask.pocoo.org/) and the JavaScript is dependent on jQuery. [Bootstrap](http://getbootstrap.com/) HTML classes are used to improve the *out-of-the-box* styling, though this is not necessary.

The template will work without JavaScript (as it is currently set-up), however, the user will be redirected to a page that displays the JSON output of the classes. The code is meant to be added around your existing Flask app, and so feel free to change the routes as necessary.

### Template Assumed Structure
```
+- app.py
+- templates/ 
|  +- tone_form.html 
+- static/ 
|  +- js/ 
|  |  +- main.js 
+- ... 
```

### app.py
```python
# ...
```

### templates/classify_form.html
```html
<!-- .. -->
```

# static/js/main.js
```javascript
// ... 
```