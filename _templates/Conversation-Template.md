---
layout: post
title:  "Conversation - Template"
img: "imgs/svg/Conversation.svg"
description: "The conversation service is a service which leverages Watson's NLP abilities to assist in the development and deployment of chatbots. It has an easy-to-use web interface which allows for the creation of conversations, all powered by Watson."
bm_url: "https://console.bluemix.net/catalog/services/conversation"
---
# Conversation
The [conversation](https://console.bluemix.net/docs/services/conversation/getting-started.html) service is a service which leverages Watson's NLP abilities to assist in the development and deployment of chatbots. It has an easy-to-use web interface which allows for the creation of conversations, all powered by Watson.

1. Generate Credentials on Bluemix
2. Create a Conversation Workspace (there is a web-based tool for this)

## The Template
This template will allow you to provide an input field for the user, have them click the submit button, and have the messages sent. Watson then returns its follow-on message, which is also showed to the user. The template assumes you are using [Flask](http://flask.pocoo.org/) and the JavaScript is dependent on jQuery. [Bootstrap](http://getbootstrap.com/) HTML classes are used to improve the *out-of-the-box* styling, though this is not necessary.

The template will work without JavaScript (as it is currently set-up), however, the user will be redirected to a page that displays the JSON output of the classes. The code is meant to be added around your existing Flask app, and so feel free to change the routes as necessary.

### Template Assumed Structure
~~~
+- app.py
+- templates/ 
|  +- classify_form.html 
+- static/ 
|  +- js/ 
|  |  +- main.js 
|  +- css/
|  |  +- main.css
+- ... 
~~~

### app.py
~~~python
import os
from watson_developer_cloud import ConversationV1
from flask import Flask, jsonify, request, render_template

# ... 

conversation = ConversationV1(
  username='<Your_Username_Here>',
  password='<Your_Password_Here>',
  version='2017-05-26')

# Workspace ID
# Workspace ID from conversation.list_workspaces() method
workspace_id = '<Your_Workspace_ID>' # Replace with your WID

# Set a global ``context'' variable for use in Conversation
global context
context = {}

# ...

@app.route('/conversation', methods=['GET','POST'])
def manage_conversation():
    # Specify to use the global context variable
    global context

    # Load Standard Module
    if request.method == 'GET':
        return render_template('conversation.html')
    
    # Validate that Message Was Submitted
    if "message" not in request.form or len(request.form["message"]) <= 0:
        return jsonify({'message': 'No message was provided.'}), 400

    # Get the response and update the context
    message_response = conversation.message(workspace_id = workspace_id, message_input = {'text': request.form['message']}, context=context)
    context = message_response["context"]

    return jsonify(message_response)
~~~

### templates/classify_form.html
~~~html
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>Natural Language Classifier Form</title>
        <!-- Include Bootstrap CSS -->
        <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-beta/css/bootstrap.min.css" integrity="sha384-/Y6pD6FV/Vv2HJnA6t+vslU6fwYXjCFtcEpHbNJ0lyAFsXTsjBbfaDjzALeQsN6M" crossorigin="anonymous">        
        <link rel="stylesheet" href="static/css/main.css">
    </head>
    <body>
        
        <div class="container mt-3" id="messages">
            <div id="messages-content">
                <p class="remove text-muted text-center">Enter a message to begin chatting with Watson...</p>
            </div>
        </div>

        <div class="container mt-0 p-0">
            <form data-id="conversation" class="ajax_form" action="conversation" method="POST">
                <div class="form-group mb-0">
                    <textarea name="message" class="form-control" placeholder="Message for Watson..."></textarea>
                </div>
                <input type="submit" value="Send!" class="btn btn-block btn-primary">
            </form>
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

# static/css/main.css
~~~css
#messages {
    height: 75vh;
    background: #e4e6e8;
    position: relative;
    overflow-y: auto;
    overflow-x: hidden;
}
#messages-content {
    position: absolute;
    bottom: 0;
    width: 100%;
}
~~~

# static/js/main.js
~~~javascript
/* 
    * parseMessage(message): Defined to handle what happens with
    *       the messages object which is returned from Watson Conversation
*/
function parseMessage(message) {
    if (! "output" in message || message['output']['text'].length <= 0) {
        // Return an Error Since the Messages Object is not well-formed
        $error_alert = $('<div class="alert alert-danger alert-dismissable fade show" role="alert" />');
        $error_alert.append('<button type="button" class="close" data-dismiss="alert" aria-label="Close"><span aria-hidden="true">&times;</span></button>');
        $error_alert.append('Watson Conversation is Experiencing an Issue Currently...');

        $("#messages").prepend($error_alert);
        return false;
    }

    // Clear any previous results
    $(".remove").remove();

    // Add the message
    $message = $('<div class="alert alert-dark col-md-7" />');
    $message.append("<strong>Watson: </strong>");
    $message.append(message['output']['text']);

    $("#messages-content").append($message);
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
            if (form_id === 'conversation') {
                // Add the submitted message to the conversation, after ensuring that "remove" are removed
                $(".remove").remove();
                $message = $('<div class="alert alert-primary col-md-7 ml-auto" />');
                $message.append("<strong>You: </strong>");
                $message.append($("textarea[name='message']").val());
                $("textarea[name='message']").val("");
                $("#messages-content").append($message);
            }
        },
        success: function(response) {
            if (form_id === "conversation") {
                // Call the parseMessage on the response
                parseMessage(response);
            }
        }
    });

    // Prevent Form Submission
    return false;
});


~~~