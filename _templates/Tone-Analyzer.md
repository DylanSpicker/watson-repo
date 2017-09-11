---
layout: post
title:  "Tone Analyzer - Template"
img: "imgs/svg/ToneAnalyzer.svg"
description: "The tone analyzer service takes in a sample of writing and conducts an assessment of the emotional, social, and linguistic tones contained within. In addition, the NLP abilities can be applied to determine specific tones or emotions which are present."
bm_url: "https://console.bluemix.net/catalog/services/tone-analyzer"
---
# Tone Analyzer
The [Tone Analyzer](https://console.bluemix.net/docs/services/tone-analyzer/getting-started.html) is a Watson Service that is used to gauge the emotional, social, and language tones found within a writing sample.

1. Generate Credentials on Bluemix

## The Template
This template will allow you to provide an input field for the user, have them click the submit button, and have the tone scores returned to the user. The template assumes you are using [Flask](http://flask.pocoo.org/) and the JavaScript is dependent on jQuery. [Bootstrap](http://getbootstrap.com/) HTML classes are used to improve the *out-of-the-box* styling, though this is not necessary.

This template also makes use of [Google Charts](https://developers.google.com/chart/), to display the results nicely to the user.

The template will work without JavaScript (as it is currently set-up), however, the user will be redirected to a page that displays the JSON output of the classes. The code is meant to be added around your existing Flask app, and so feel free to change the routes as necessary.

### Template Assumed Structure
~~~
+- app.py
+- templates/ 
|  +- tone_analyzer_form.html 
+- static/ 
|  +- js/ 
|  |  +- main.js 
+- ... 
~~~

### app.py
~~~python
from flask import Flask
from flask import render_template
from flask import request
from flask import jsonify
from watson_developer_cloud import ToneAnalyzerV3

# ...

tone_analyzer = ToneAnalyzerV3(version='2016-05-19',
                               username='<Your_Tone_Analyzer_Username>',
                               password='<Your_Tone_Analyzer_Password>')

# ...

@app.route('/tone-analyzer', methods=['GET', 'POST'])
def handle_tone_analyzer():
    if request.method == 'GET':
        return render_template('tone_analyzer_form.html', sentences=False)

    tone_request = "" if "tone_text" not in request.form.keys() else request.form['tone_text']
    tones = {} if tone_request == "" else tone_analyzer.tone(tone_request)

    return jsonify(tones)

~~~

### templates/tone\_analyzer\_form.html
~~~html
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>Tone Analyzer Form</title>
        <!-- Include Bootstrap CSS -->
        <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-beta/css/bootstrap.min.css" integrity="sha384-/Y6pD6FV/Vv2HJnA6t+vslU6fwYXjCFtcEpHbNJ0lyAFsXTsjBbfaDjzALeQsN6M" crossorigin="anonymous">        
    </head>
    <body>
        
        <div class="container mt-3" id="messages"></div>

        <div class="container mt-5">
            <form data-id="tone-analyzer" class="ajax_form" action="tone-analyzer" method="POST">
                <div class="form-group mb-0">
                    <textarea name="tone_text" class="form-control" placeholder="Text to Analyze..."></textarea>
                </div>
                <input type="submit" value="Analyze!" class="btn btn-block btn-primary">
            </form>
            <div id="results" class="mt-5 row">
                <div class="col-4">
                    <div id="emotion_tone_holder"></div>
                </div>
                <div class="col-4">
                    <div id="language_tone_holder"></div>
                </div>
                <div class="col-4">
                    <div id="social_tone_holder"></div>
                </div>
            </div>
        </div>

        <!-- Bootstrap and jQuery JavaScript Files -->
        <script src="https://code.jquery.com/jquery-3.2.1.min.js" ></script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.11.0/umd/popper.min.js" integrity="sha384-b/U6ypiBEHpOf/4+1nzFpr53nxSS+GLCkfwBdFNTxtclqqenISfwAzpKaMNFNmj4" crossorigin="anonymous"></script>
        <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-beta/js/bootstrap.min.js" integrity="sha384-h0AbiXch4ZDo7tp9hKZ4TsHbi047NrKGLO3SEJAg45jXxnGIfYzk4Si90RDIqNm1" crossorigin="anonymous"></script>
        <!-- Google Graphing Tools -->
        <script type="text/javascript" src="https://www.gstatic.com/charts/loader.js"></script>        
        <!-- Custom JS File -->
        <script src="static/js/main.js"></script>
    </body>
</html>
~~~

# static/js/main.js
~~~javascript
$(document).ready(function(){
    // Load in the Google Charts packages
    google.charts.load('current', {packages: ['corechart', 'bar', 'treemap']});    
});

/*
    * addToneGraphs(tones): Take the watson tone object and add the three
    * corresponding graphs to the page.
*/
function addToneGraphs(tones) {
    
    if ("document_tone" in tones) {
        
        // Create the Emotion Tone Bar Chart
        
        var emotion_tone_data = [['Tone', 'Tone Score']];
        tones["document_tone"]["tone_categories"][0]["tones"].forEach(function(el, index){
            emotion_tone_data.push([el["tone_name"], el["score"]]);
        });
        var emotion_tone_graph = google.visualization.arrayToDataTable(emotion_tone_data),
            materialChart_emotion = new google.charts.Bar(document.getElementById('emotion_tone_holder'));

        materialChart_emotion.draw(emotion_tone_graph, {chart: {title: "Emotional Tone Content"}, colors: ['#F44336'], legend: {position: 'none'}, bars: 'horizontal'});

        // Create the Language Tone Bar Chart
        
        var language_tone_data = [['Tone', 'Tone Score']];
        tones["document_tone"]["tone_categories"][1]["tones"].forEach(function(el, index){
            language_tone_data.push([el["tone_name"], el["score"]]);
        });
        var language_tone_graph = google.visualization.arrayToDataTable(language_tone_data),
            materialChart_language = new google.charts.Bar(document.getElementById('language_tone_holder'));

        materialChart_language.draw(language_tone_graph, {chart: {title: "Language Tone Content"}, colors: ['#3F51B5'], legend: {position: 'none'}});

        // Create the Social Tone Bar Chart
        
        var social_tone_data = [['Tone', 'Tone Score']];
        tones["document_tone"]["tone_categories"][2]["tones"].forEach(function(el, index){
            social_tone_data.push([el["tone_name"], el["score"]]);
        });
        var social_tone_graph = google.visualization.arrayToDataTable(social_tone_data),
            materialChart_social = new google.charts.Bar(document.getElementById('social_tone_holder'));

        materialChart_social.draw(social_tone_graph, {chart: {title: "Social Tone Content"}, colors: ['#673AB7'], legend: {position: 'none'}, bars: 'horizontal'});
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
            if (form_id === "tone-analyzer") {
                // Add a message that the form is being submitted
                $submission_alert = $('<div class="alert alert-info alert-dismissable fade show" role="alert" />');
                $submission_alert.append('<button type="button" class="close" data-dismiss="alert" aria-label="Close"><span aria-hidden="true">&times;</span></button>');
                $submission_alert.append('The text is being analyzed!');

                $("#messages").append($submission_alert);
            }
        },
        success: function(response) {
            if (form_id === "tone-analyzer") {
                // Call the addToneGraphs function on the response
                addToneGraphs(response);
            }
        }
    });

    // Prevent Form Submission
    return false;
});
~~~