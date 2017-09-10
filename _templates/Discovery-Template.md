---
layout: post
title:  "Discovery - Template"
img: "imgs/svg/Discovery.svg"
---
# Natural Language Classifier
The [natural language classifier](https://console.bluemix.net/docs/services/natural-language-classifier/getting-started.html) (NLC) is a Watson Service that is used to classify snippets of text based on classes that you define. In order to use the classifier you must:

1. Generate Credentials on Bluemix
2. Create and Train a Classifier (there is a web-based tool for this)

## The Template
This template will allow you to provide an input field for the user, have them click the submit button, and have the predicted classes (along with their confidence scores) returned to the user. The template assumes you are using [Flask](http://flask.pocoo.org/) and the JavaScript is dependent on jQuery. [Bootstrap](http://getbootstrap.com/) HTML classes are used to improve the *out-of-the-box* styling, though this is not necessary.

The template will work without JavaScript (as it is currently set-up), however, the user will be redirected to a page that displays the JSON output of the classes. The code is meant to be added around your existing Flask app, and so feel free to change the routes as necessary.

### Template Assumed Structure
~~~
+- app.py
+- templates/ 
|  +- classify_form.html 
+- static/ 
|  +- js/ 
|  |  +- main.js 
+- ... 
~~~

### app.py
~~~python
import os
from watson_developer_cloud import NaturalLanguageClassifierV1
from flask import jsonify
from flask import request
from flask import render_template

# ... 

natural_language_classifier = NaturalLanguageClassifierV1(
  username='<NLC_Username>',
  password='<NLC_Password>')
classifier_id = '<Classifier_ID>'

# ...

@app.route('/classify', methods=['GET','POST'])
def handle_classification():
    if request.method == 'GET':
        # Return the HTML Form
        return render_template('classify_form.html')
    
    # Set the comment_text to the comment_text form variable 
    # if it exists
    comment_text = "" if "comment_text" not in request.form.keys() else request.form['comment_text']

    # Call the classify method if comment_text is not blank
    classes = {} if comment_text == "" else natural_language_classifier.classify(classifier_id, comment_text)

    return jsonify(classes)
~~~

### templates/classify_form.html
~~~html
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>Natural Language Classifier Form</title>
        <!-- Include Bootstrap CSS -->
        <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-beta/css/bootstrap.min.css" integrity="sha384-/Y6pD6FV/Vv2HJnA6t+vslU6fwYXjCFtcEpHbNJ0lyAFsXTsjBbfaDjzALeQsN6M" crossorigin="anonymous">        
    </head>
    <body>
        
        <div class="container mt-3" id="messages"></div>

        <div class="container mt-5">
            <form data-id="nlc" class="ajax_form" action="classify" method="POST">
                <div class="form-group mb-0">
                    <textarea name="comment_text" class="form-control" placeholder="Comment to Analyze..."></textarea>
                </div>
                <input type="submit" value="Analyze!" class="btn btn-block btn-primary">
            </form>
            <ul id="results_list" class="mt-3 list-group"></ul>
        </div>

        <!-- Bootstrap and jQuery JavaScript Files -->
        <script src="https://code.jquery.com/jquery-3.2.1.min.js" ></script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.11.0/umd/popper.min.js" integrity="sha384-b/U6ypiBEHpOf/4+1nzFpr53nxSS+GLCkfwBdFNTxtclqqenISfwAzpKaMNFNmj4" crossorigin="anonymous"></script>
        <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-beta/js/bootstrap.min.js" integrity="sha384-h0AbiXch4ZDo7tp9hKZ4TsHbi047NrKGLO3SEJAg45jXxnGIfYzk4Si90RDIqNm1" crossorigin="anonymous"></script>
        <!-- Custom JS File -->
        <script src="static/js/main.js"></script>
    </body>
</html>
~~~

# static/js/main.js
~~~javascript
/* 
    * parseClasses(classes): Defined to handle what happens with
    *       the classes object which is returned from Watson
*/
function parseClasses(classes) {
    if (! "classes" in classes || classes['classes'].length <= 0) {
        // Return an Error Since the Classes Object is not well-formed
        $error_alert = $('<div class="alert alert-danger alert-dismissable fade show" role="alert" />');
        $error_alert.append('<button type="button" class="close" data-dismiss="alert" aria-label="Close"><span aria-hidden="true">&times;</span></button>');
        $error_alert.append('The comment is being analyzed!');

        $("#messages").append($error_alert);
        return false;
    }
    // Clear any previous results
    $("#results_list").html("");

    // Add the list items
    classes['classes'].forEach(function(element) {
        $li = $("<li class='list-group-item list-group-item-action'></li>");
        $li.html(element['class_name'] + " (" + (parseFloat(element['confidence'])*100).toFixed(2) + "%)");
        $("#results_list").append($li);
    });
}

/* 
    * The following code submits any arbitrary form with the class ajax_form
    * to the endpoint, with the method that is specified. In this sense, you 
    * can easily add this on to improve the functioning of an HTML form that 
    * you get to work without the jQuery;
    * In this case the success is programmed directly, given the data-attribute of
    * the form. This logic could be extracted outside of the success call relatively
    * easily.
*/ 
$(".ajax_form").on("submit", function(){
    var url = $(this).attr("action"),
        method = $(this).attr("method"),
        data = $(this).serializeArray(),
        form_id = $(this).data("id");

    $.ajax({
        url: url,
        method: method,
        data: data,
        beforeSend: function() {
            if (form_id === "nlc") {
                // Add a message that the form is being submitted
                $submission_alert = $('<div class="alert alert-info alert-dismissable fade show" role="alert" />');
                $submission_alert.append('<button type="button" class="close" data-dismiss="alert" aria-label="Close"><span aria-hidden="true">&times;</span></button>');
                $submission_alert.append('The comment is being analyzed!');

                $("#messages").append($submission_alert);
            }
        },
        success: function(response) {
            if (form_id === "nlc") {
                // Call the parseClasses on the response
                parseClasses(response);
            }
        }
    });

    // Prevent Form Submission
    return false;
});
~~~