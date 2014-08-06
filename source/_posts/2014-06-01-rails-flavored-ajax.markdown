---
layout: post
title: "Rails Flavored Ajax"
date: 2014-06-01 18:28:07 -0700
comments: true
categories: [Rails, Ajax]
keywords: Ruby on Rails, Tealeaf Academy, Ajax, SJR
description: Server-Generated Javascript Response (SJR) - How Rails implements Ajax
---

####Unobtrusive Javascript to Handle Ajax

Ajax stands for Asynchronous Javascript and XML and is a way of sending and retrieving data from a server asynchronously. This is particularly useful when we have a page that is very expensive to load, and we want to update a small portion of the UI without having to reload the entire page from scratch. The basic ajax pattern consists of 3 parts:

- Unobtrusive javascript event listener (eg. `.click`)
- Triggering Ajax request
- Handling the response

First, there must be some type of event handler that initiates the Ajax request, such as clicking on a link or submitting a form. Then, the request is sent to the server, and finally the response is generated and sent back to the browser. Unobtrusive javascript or jQuery has been used traditionally for handling Ajax calls. These are known as *client side code* because they live on the client-side, or on the browser. An example of such code is shown below where we ajaxify the submission of a form.  

```javascript Ex1: Ajax with Unobtrusive Javascript
$(document).ready(function(){
  $('form#new_post').submit(function(e){
    $.ajax({
      type: 'POST',
      url: '/posts',
      data: { title: $('#post_title').val(),
              url: $('#post_url').val(),
              description: $('#post_description').val()
      },
    }).done(function(msg) {
      // callback function 
    })
    e.preventDefault();
  });
});
```

In this example, we have a model-backed form with id `new_post` with the fields `title`, `url`, and `description`. The `.submit` is the event handler that triggers the Ajax request, and `.done` is the callback function that handles the response. The `data` (corresponding to the fields of the form) is wrapped in a hash and sent to the server. This method works perfectly fine, but notice how cumbersome the process is - we have to manually traverse the DOM and target the HTML elements (i.e. the event handler and the data parameters), every time we wish to issue a different Ajax call. Repeating this process can be time consuming and a bit annoying. 

####How Rails Implements Ajax

Rails drastically simplifies the implementation of Ajax. The Rails syntax for ajaxifying an element is by adding a `remote: true` attribute. For example, if we want to ajaxify a form submission as before, we would simply say `<%= form_for @post, remote: true do |f| %>`. The `remote: true` adds a `data-remote=true` attribute to the HTML element, which Rails' built-in javascript file uses to convert the form into an Ajax call. This step essentially generates a structure similar to the code in `Ex1`, and submits the request along with the data parameters. 

We can then handle the request inside the `posts#create`, the controller action that handles the form submission. 

```ruby Ex2: Handling the Ajax Request 
def create
  @post = Post.new(post_params) 

  respond_to do |format|
    format.html do 
      if @post.save
        flash[:notice] = "You successfully created a new post"
        redirect_to posts_path
      else
        render :new 
      end
    end
    format.js do 
      @post.save
    end
  end
end
```   
Since `remote: true` ajaxified the form, the client is expecting a javascript response. Rails segregates the response based on the format (eg: html, js, json, html) by using the `respond_to` method. The block inside `format.html` will be executed if the client wants HTML in response to the action; whereas, the block inside `format.js` will be executed for a javascript or Ajax response. As in any other controller action, Rails will by default render a template with the same name as the controller action (ie. `/posts/create.js.erb`). Inside this template, we can then write unobtrusive javascript that is then sent back to the browser.

The beauty of this template is that it lives on the *server side*, meaning we can write ruby code within these templates and access instance variables and other server-side data. This enables us to write code such as

```javascript Ex3: Inside .js.erb Template
alert("The title of the post is: <%= @post.title %>");
```
where we are accessing the `@post` object and its attribute that live on the server. This is why Rails flavored Ajax is dubbed *Server-Generated Javascript Response (SJR)* - the javascript response is generated strictly on the server side, as opposed to unobtrusive javascript shown in `Ex1`, which handles the response on the client side using a callback function. 

####Summmary
The Rails way of implementing Ajax is a lot less manual and faster than the traditional jQuery method, but it's important to know that under the hood, the two methods are nearly identical. That is, by adding a `data-remote=true` attribute to an HTML element, Rails uses built-in javascript to convert the element into an Ajax call, which essentially builds the jQuery code from `Ex1` for us. We can then handle the response by rendering a `js.erb` file, where we have access to data and instance variables living on the server. The embedded ruby - javascript code is then converted into pure javascript and sent back to the browser. 





