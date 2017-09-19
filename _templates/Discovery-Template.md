---
layout: post
title:  "Discovery - Template"
img: "imgs/svg/Discovery.svg"
description: "The discovery service provides a rich corpus annotation and natural language search. It ingests text documents, applies a NLP model (pre-trained or custom) and then allows for queries to be made of the corpus in a natural language tone."
bm_url: "https://console.bluemix.net/catalog/services/discovery"
---
# Discovery Template
The [Discovery](https://console.bluemix.net/docs/services/discovery/getting-started.html) service is a service which leverages Watson's NLP abilities to assist in the development and deployment of rich text search and QA applications. It has an easy-to-use web interface which allows for the creation of a corpus, all powered by Watson.

1. Generate Credentials on Bluemix
2. Create a Corpus (or use Discovery News)

## The Template
This template will allow you to provide an input field for the user, have them click the submit button, and have the messages sent. Watson then returns its follow-on message, which is also showed to the user. The template assumes you are using [Flask](http://flask.pocoo.org/) and the JavaScript is dependent on jQuery. [Bootstrap](http://getbootstrap.com/) HTML classes are used to improve the *out-of-the-box* styling, though this is not necessary.

The template will work without JavaScript (as it is currently set-up), however, the user will be redirected to a page that displays the JSON output of the classes. The code is meant to be added around your existing Flask app, and so feel free to change the routes as necessary.

### Template Assumed Structure
~~~
+- app.py
+- templates/ 
|  +- discovery.html 
+- static/ 
|  +- js/ 
|  |  +- main.js
+- ... 
~~~

### app.py
~~~python
import os
from watson_developer_cloud import DiscoveryV1
from flask import Flask, jsonify, request, render_template

# ...
discovery = DiscoveryV1(
  username='<Your_Discovery_Username>',
  password='<Your_Discovery_Password>',
  version='2017-07-19'
)

collection_id      = "<Your_Collection_ID>"
enivornment_id     = "<Your_Environment_ID>"
# ...

@app.route('/discovery', methods=['GET','POST'])
def manage_discovery():
    
    # Load Standard Module
    if request.method == 'GET':
        return render_template('discovery.html')
    
    # Validate that Query Was Submitted
    if "query" not in request.form or len(request.form["query"]) <= 0:
        return jsonify({'message': 'No search query was provided.'}), 400

    # Specify the options for the query (more details in API Reference)
    qopts = {'natural_language_query': request.form['query'], 'passages': True}
    
    # Run the Discovery Query + Fetch Response
    query_response = discovery.query(enivornment_id, collection_id, qopts)

    return jsonify(query_response)

~~~

### templates/discovery.html
~~~html
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>Discovery Template</title>
        <!-- Include Bootstrap CSS -->
        <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-beta/css/bootstrap.min.css" integrity="sha384-/Y6pD6FV/Vv2HJnA6t+vslU6fwYXjCFtcEpHbNJ0lyAFsXTsjBbfaDjzALeQsN6M" crossorigin="anonymous">        
    </head>
    <body>
        
        <div class="container mt-3" id="messages"></div>

        <div class="container mt-0 p-0">
            <form data-id="discovery" class="ajax_form" action="discovery" method="POST">
                <div class="form-group mb-5">
                    <div class="input-group">
                        <input type="text" name="query" class="form-control" placeholder="Search for..." />
                        <span class="input-group-btn">
                            <input type="submit" value="Search!" class="btn btn-primary">
                        </span>
                    </div>
                </div>
            </form>

            <ul id="results" class="list-group"></ul>
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
    * parseDiscovery(discovery_object): Defined to handle what happens with
    *       the discovery object which is returned from Watson Discovery
*/
function parseDiscovery(discovery_object) {
    if (! "results" in discovery_object || discovery_object['results'].length <= 0) {
        // Return an Error Since the Messages Object is not well-formed
        $error_alert = $('<div class="alert alert-danger alert-dismissable fade show" role="alert" />');
        $error_alert.append('<button type="button" class="close" data-dismiss="alert" aria-label="Close"><span aria-hidden="true">&times;</span></button>');
        $error_alert.append('Watson Discovery is Experiencing an Issue Currently...');

        $("#messages").prepend($error_alert);
        return false;
    }

    // Clear Previous Results
    $("#results").html("");

    // Iterate through the Results
    discovery_object["results"].forEach(function(result, index){
        list_class = '';
        href = '#';

        // Assign colour based on sentiment
        if ("docSentiment" in result && "type" in result["docSentiment"]) {
            if (result["docSentiment"]["type"] == "positive") {
                list_class = 'list-group-item-success';
            } else if (result["docSentiment"]["type"] == "neutral") {
                list_class = 'list-group-item-warning';
            } else if (result["docSentiment"]["type"] == "negative") {
                list_class = 'list-group-item-danger';
            }
        } 

        // Pull out href if it exists
        if ("url" in result) {
            href = result["url"];
        }

        $("#results").append(
            "<a target='_BLANK' class='list-group-item list-group-item-action "+list_class+"' href='"+href+"'>" + 
            result["title"] + 
            "</a>");
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
            // No before send behaviour necessary
        },
        success: function(response) {
            if (form_id === "discovery") {
                // Call the parseMessage on the response
                parseDiscovery(response);
            }
        }
    });

    // Prevent Form Submission
    return false;
});
~~~