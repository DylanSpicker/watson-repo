---
layout: post
title:  "Personality Insights - Template"
date:   2017-08-23 08:40:39 -0400
categories: jekyll update
---
# Natural Language Classifier
The [Personality Insights](https://console.bluemix.net/docs/services/personality-insights/getting-started.html) is a Watson Service that is used to generate a personality profile, including the Big 5 description, needs, values, and consumer preferences. In order to use the classifier you must:

1. Generate Credentials on Bluemix for Persoanlity Insights

## The Template
This template will allow you to provide an input field for the user, have them click the submit button, and have the generated profile returned to the user. The template assumes you are using [Flask](http://flask.pocoo.org/) and the JavaScript is dependent on jQuery. [Bootstrap](http://getbootstrap.com/) HTML classes are used to improve the *out-of-the-box* styling, though this is not necessary.

The template will work without JavaScript (as it is currently set-up), however, the user will be redirected to a page that displays the JSON output of the classes. The code is meant to be added around your existing Flask app, and so feel free to change the routes as necessary.

### Template Assumed Structure
~~~
+- app.py
+- templates/ 
|  +- insights_form.html 
+- static/ 
|  +- js/ 
|  |  +- main.js 
+- ... 
~~~

### app.py
~~~python
import os
from watson_developer_cloud import PersonalityInsightsV3
from flask import jsonify
from flask import request
from flask import render_template

# ...

personality_insights = PersonalityInsightsV3(
                            version='2016-10-20',
                            username='<PI_Username>',
                            password='<PI_Password>')

# ...

@app.route('/personality-insights', methods=['GET','POST'])
def analyze_personality():
    if request.method == 'GET':
        # Return the HTML Form
        return render_template('insights_form.html')

    # Set the profile_text variable to be the user_profile form variable
    # if it exists
    profile_text = "" if "user_profile" not in request.form.keys() else request.form['user_profile']

    profile = {} if profile_text == "" else personality_insights.profile(profile_text, consumption_preferences=True)

    return jsonify(profile)
~~~

### templates/insights_form.html
~~~html
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>Personality Insights Form</title>
        <!-- Include Bootstrap CSS -->
        <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-beta/css/bootstrap.min.css" integrity="sha384-/Y6pD6FV/Vv2HJnA6t+vslU6fwYXjCFtcEpHbNJ0lyAFsXTsjBbfaDjzALeQsN6M" crossorigin="anonymous">        
    </head>
    <body>
        
        <div class="container mt-3" id="messages"></div>

        <div class="container mt-5">
            <form data-id="personality-insights" class="ajax_form" action="personality-insights" method="POST">
                <div class="form-group mb-0">
                    <textarea name="user_profile" class="form-control" placeholder="Profile to Analyze..."></textarea>
                </div>
                <input type="submit" value="Analyze!" class="btn btn-block btn-primary">
            </form>
            <div id="results" class="mt-5"></div>
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
    * generateProfile(profile): Defined to handle the output of the 
    *       personality profile from the PI Watson Service
*/
function generateProfile(profile) {

    if ("personality" in profile) {
        // Loop through the Personality Elements
        $("#results").append("<h2>Big 5 Personality</h2>");
        $("#results").append("<hr>");
        profile['personality'].forEach(function(trait){

            $row = $("<div class='row' />");
            $col_6_1 = $("<div class='col-2'>");
            $col_6_2 = $("<div class='col-10'>");

            $label = $('<p></p>');
            $label.html(trait['name']);

            $progress_bar_container = $('<div class="progress mt-1 mb-1">');
            $bar = $('<div class="progress-bar" role="progressbar" style="width: '+
                                                    (trait['percentile']*100).toFixed(2)+'%; height: 25px;" aria-valuenow="'
                                                    +(trait['percentile']*100).toFixed(2)+'" aria-valuemin="0" aria-valuemax="100" />');
            $bar.html((trait['percentile']*100).toFixed(2)+"%");
            $bar.appendTo($progress_bar_container);
            
            $col_6_1.append($label);
            $col_6_2.append($progress_bar_container);
            $row.append($col_6_1);
            $row.append($col_6_2);
            
            $("#results").append($row);
        });
    }
    if ("needs" in profile) {
        // Loop through the 'needs' elements
        $("#results").append("<h2 class='mt-3'>Needs</h2>");
        $("#results").append("<hr>");
        profile['needs'].forEach(function(trait){

            $row = $("<div class='row' />");
            $col_6_1 = $("<div class='col-2'>");
            $col_6_2 = $("<div class='col-10'>");

            $label = $('<p></p>');
            $label.html(trait['name']);

            $progress_bar_container = $('<div class="progress mt-1 mb-1">');
            $bar = $('<div class="bg-success progress-bar" role="progressbar" style="width: '+
                                                    (trait['percentile']*100).toFixed(2)+'%; height: 25px;" aria-valuenow="'
                                                    +(trait['percentile']*100).toFixed(2)+'" aria-valuemin="0" aria-valuemax="100" />');
            $bar.html((trait['percentile']*100).toFixed(2)+"%");
            $bar.appendTo($progress_bar_container);
            
            $col_6_1.append($label);
            $col_6_2.append($progress_bar_container);
            $row.append($col_6_1);
            $row.append($col_6_2);
            
            $("#results").append($row);
        });
    }
    if ("values" in profile) {
        // Loop through the 'values' elements
        $("#results").append("<h2 class='mt-3'>Values</h2>");
        $("#results").append("<hr>");
        profile['values'].forEach(function(trait){

            $row = $("<div class='row' />");
            $col_6_1 = $("<div class='col-2'>");
            $col_6_2 = $("<div class='col-10'>");

            $label = $('<p></p>');
            $label.html(trait['name']);

            $progress_bar_container = $('<div class="progress mt-1 mb-1">');
            $bar = $('<div class="bg-info progress-bar" role="progressbar" style="width: '+
                                                    (trait['percentile']*100).toFixed(2)+'%; height: 25px;" aria-valuenow="'
                                                    +(trait['percentile']*100).toFixed(2)+'" aria-valuemin="0" aria-valuemax="100" />');
            $bar.html((trait['percentile']*100).toFixed(2)+"%");
            $bar.appendTo($progress_bar_container);
            
            $col_6_1.append($label);
            $col_6_2.append($progress_bar_container);
            $row.append($col_6_1);
            $row.append($col_6_2);
            
            $("#results").append($row);
        });
    }
    if ("consumption_preferences" in profile) {
        // Loop through the consumption preferences
        $("#results").append("<h2 class='mt-3'>Consumption Preferences</h2>");
        $("#results").append("<hr>");
        profile['consumption_preferences'].forEach(function(trait){
            $pref_list = $("<ul class='list-group mt-2 mb-2'>");
            $pref_list.append("<li class='list-group-item active'>"+trait['name']+"</li>");

            trait['consumption_preferences'].forEach(function(pref){
                $li = $("<li class='list-group-item'>");
                $li.html(pref['name']);

                if (pref['score'] == 0) {
                    $li.addClass('list-group-item-danger');
                } else if (pref['score'] == 1) {
                    $li.addClass('list-group-item-success');
                } else {
                    $li.addClass('list-group-item-warning');
                }

                $pref_list.append($li);
            });
            
            $("#results").append($pref_list);
        });
    }
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
            if (form_id === "personality-insights") {
                // Add a message that the form is being submitted
                $submission_alert = $('<div class="alert alert-info alert-dismissable fade show" role="alert" />');
                $submission_alert.append('<button type="button" class="close" data-dismiss="alert" aria-label="Close"><span aria-hidden="true">&times;</span></button>');
                $submission_alert.append('The user profile is being analyzed!');

                $("#messages").append($submission_alert);
            }
        },
        success: function(response) {
            if (form_id === "personality-insights") {
                // Call the generateProfile function on the response
                generateProfile(response);
            }
        }
    });

    // Prevent Form Submission
    return false;
});


~~~