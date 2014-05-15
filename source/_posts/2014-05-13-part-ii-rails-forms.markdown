---
layout: post
title: "Part II: Rails Forms"
date: 2014-05-13 22:41:29 -0700
comments: true
categories: [Rails, Forms, Strong Parameters, Validation]
---

In the last post, we discussed how to generate forms, how the parameters are stored inside a `params` hash, and where the form is submitted to. In this post, we talk about how to handle the form submission using validations and strong parameters. 

####Strong Parameters

Strong parameters is a security feature that protects attributes from end-user mass assignment. Quoting the [Rails API](http://api.rubyonrails.org/classes/ActionController/StrongParameters.html), 

> "With strong parameters, Action Controller parameters are forbidden to be used in Active Model mass assignments until they have been whitelisted.

This means that we manually specify which attributes are allowed for mass updating. To whitelist an entire hash of parameters, the `permit!` method can be used: `params.require(:post).permit!`. The `require(:post)` suggests that the attributes are nested inside a `post` top-level key (which as we recall, is what we get using model-backed forms). 

####Validations

Next, validations are used to ensure that only valid data is saved into our database. For example, if every post must have `title`, `url`, and `description` attributes for the post to be valid, we must explicitly specify the validation on the *Model Layer*. Such validations might look like this:

```ruby Ex 1: Post Model Validation
class Post < ActiveRecord::Base
  validates :title, presence: true, length: { minimum: 5 }
  validates :url, presence: true, uniqueness: true
  validates :description, presence: true
end
```

An important aspect of validation is that the errors are not triggered until the changes hit the database. This means that when creating a new `post` object, the validation errors will kick in after a `.create` `.update` or `.save` methods are called on `Post` (but not `.new`). If validations are successful, the assignment will return true, and otherwise, it will return false. 

####Validation Pattern

When creating the `create` / `update` actions from a model-backed form, we use a generalized pattern to handle the validations. 

```ruby Ex 2: Validation Pattern (app/controllers/posts_controller.rb)
def create
  @post = Post.new(post_params)

  if @post.save
    flash[:notice] = "You successfully created a new post."
    redirect_to posts_path
  else
    render :new
  end
end

def update
  @post = Post.find(params[:id])

  if @post.update(post_params)
    flash[:notice] = "You successfully updated an existing post."
    redirect_to posts_path
  else
    render :edit
  end
end

private

def post_params
  params.require(:post).permit(:title, :url, :description)
end
```

There are a variety of interesting things going on here. First, we defined a private method called `post_params` which sets up the strong parameters as we discussed earlier. Inside the `create` and `update` actions, we call `post_params` to mass assign parameters to the `@post` object. Second, if the validation is successful, we `redirect_to` to `posts_path`. We can actually redirect to wherever we desire, but in this case, we are redirecting to `posts#index` action where we display all posts. Lastly and perhaps most importantly, if the validation is unsuccessful, we `render` the form template. This is VERY important!! We CANNOT redirect because if validation fails, we want to display the error messages in the `view`. This is only possible when we render because the error message is stored inside an instance variable `@post.errors`. If we redirect, we lose access to that instance variable because we're issuing another HTTP request (HTTP is a *stateless* protocol).

The `.errors` method will return a `@messages` hash that contains all validation error messages. We can extract these messages into an array-like structure using the `.errors.full_messages` method. We can then iterate through these messages and display them inside our view template:

```erb Ex 3: Displaying Validation Errors
<%= if @post.errors.any? %>
  <h5>There were some errors:</h5>
  <ul>
    <% @post.errors.full_messages.each do |msg| %>
      <li><%= msg %></li>
    <% end %>
  </ul>
<% end %>
```   

Notice here that we are using the `@post` object. This is again why it is important that we render (instead of redirect) when validation fails, because we need to access the `@post` instance variable and its associated `.errors`. 

####Summary

Strong parameters and validations work hand-in-hand within the generalized validation pattern. Rails form is one of the fundamental concepts of the framework and should not be taken for granted. As a Rails developer, it's VERY important to be able to construct and handle forms with ease. You should be able to do it blindfolded, with your hands tied behind your back, and in your sleep. It's THAT important!!!



















