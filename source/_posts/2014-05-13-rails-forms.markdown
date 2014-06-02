---
layout: post
title: "Rails Forms - Part I"
date: 2014-05-13 19:55:35 -0700
comments: true
categories: [Rails, Forms]
---

In general, there are 3 ways to create and handle forms in Rails: pure html, form helpers, and model-backed form helpers. 

####Pure HTML Way

The pure html way is just that - pure HTML. So it really has nothing to do with Rails. This approach is shown in the code below:

```html Ex 1: Form Using Pure HTML
<form action='/posts' method='POST'>
  Title: <input typ='text' name='my_title'>
  <input type='submit'>
</form>
```

The `action` and `method` attributes specify the URL and HTTP verb used for the form submission. In other words, the form will be routed to `posts#create`, which is the action in which the form submission data is parsed and processed to create a new `post` object. It is important to specify the `name` attribute inside `input` because it acts as the key that maps to the value entered in the text field (eg: `"my_title" => "some title"`). We can then recover that value by using the params hash like so: `params[:my_title]`. 

Submitting this form will actually result in an `InvalidAuthenticityToken Error` because of a security issue with cross-site scripting. Rails requires us to verify that the form was submitted from a page we generated, which is accomplished by the presence of an `authenticity_token`. Since the pure html way does not have an authenticity token built in, we should almost always avoid using this method to handle form submission. 

####Rails Non-Model Backed Form Helpers

The second method is using Non-Model Backed Forrm Helpers. The syntax is shown below:

```erb Ex 2: Form Using Non-Model Backed Form Helpers
<%= form_tag '/posts' do %>
  <%= label_tag :title %>
  <%= text_field_tag :title %>
  <br/>
  <%= submit_tag "Create Post", class: 'btn btn-primary'%>
<% end %>  
```

The above code will generate an html like this:

```html Ex 3: HTML Generated Using Rails Form Helperes
<div style="margin:0;padding:0;display:inline">
  <input name="utf8" type="hidden" value="✓">
  <input name="authenticity_token" type="hidden" value="d/fqhgShsPjPL+Mxslr6fwj4rCbLVMkndsJ1M3DJPiY=">
</div>
<label for="title">Title</label>
<input id="title" name="title" type="text">
<input class="btn btn-primary" name="commit" type="submit" value="Create Post">
```

As before, we have a `name` attribute that maps to the value submitted in the text field. But notice we also have a `div` nestsed inside the `form` element. Inside this `div`, we have a hidden `authenticity_token` that allows us to get around the Cross-Site Requeset Forgery (CSRF) protection that comes with Rails. 

####Rails Model-Backed Form Helpers

Lastly, we have the model-backed form helpers. These are used when performing "CRUD" actions on a resource (i.e. ActiveRecord model); whereas, non-model backed forms are used everywhere else where ActiveRecord models are not involved. The syntax is shown below:

```erb Ex 4: Form Using Model-Backed Form Helpers
<%= form_for @post do |f| %>
  <%= f.label :title %>
  <%= f.text_field :title %>
  <%= f.label :description %>
  <%= f.text_area :description %>
  <%= f.submit "Create Post", class: "btn btn-primary" %>
<% end %>
```

There are a couple of interesting (and confusing) conventions at work that makes model-backed forms so useful. First, the `form_for` takes in a model object instead of a url - and this is where Rails magic kicks in. If the object is a new object, Rails will send the form to the `posts#create` action; whereas, if the object is an existing one, Rails will send the form to the `posts#update` action. In other words, Rails will automatically generate the correct URL and HTTP verb depending on the model object that is passed in to the form. This may sound trivial at first, but this implies that we can use the same form for two different controller actions - one for `new` where we pass in a new `post` object, and one for `edit` where we pass in an existing `post` object. 

The second interesting feature is that the input element has a name of `post[title]`, meaning the parameters are nested inside a top-level key of `post`, as shown here: `"post" => {"title": "some title", "description": "some description"}`. We can then do something like this, `Post.new(params[:post])` to mass assign all attributes values to create a new post. 

Lastly, as the name "model-backed" implies, we can ONLY refer to *Attributes* or *Virtual Attributes* available to us by the ActiveRecord model. Therefore, we cannot use arbitrary keys that do not reside in our database. For example, `<%= f.text_field :whatever %>` will not work because `whatever` is not a getter/setter that is made available to us by the `Post` model. This is also why model-backed helpers are used when acting on a resource, and non-model backed helpers are used everywhere else.     

####Summary

To summarize the above in one sentence: never use pure HTML forms, use model-backed forms when working with an ActiveRecord model, and use non model-backed forms for all other occassions. In the next post, we will discuss how to handle the actual submission of the form using strong parameters and validations. 







