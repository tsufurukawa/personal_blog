---
layout: post
title: "Authentication - Part II"
date: 2014-05-22 21:54:20 -0700
comments: true
categories: [Rails, Authentication]
keywords: Ruby on Rails, Tealeaf Academy, authentication, sessions
description: Simulating 'login' and 'logout' using authentication and sessions
---

In this post, we'll simulate a `login` and `logout` user interface by introducing a concept called `session`. A user's session is tracked using a cookie, which exists only in temporary memory while the user is logged in and navigating the website. When the user logs-out, the session cookie is then destroyed.

####Simulating the Login

Before we create a form allowing users to login, we'll more than likely have to create routes that can handle the request. Now conceptually, we might at first be tempted to simulate login/logout using the `users` resource, but this is incorrect because we are not creating/updating/deleting, or performing CRUD actions, on a user object. Instead we need a resource that keeps track of the state of whether a user is logged-in or not. We will accomplish this using a `SessionsController` (the name is arbitrary), and we'll first manually define routes as following:

```ruby Ex1: Building Resources Routes
get '/login', to: 'sessions#new'
post '/login', to: 'sessions#create'
get '/logout', to: 'sessions#destroy'
```

Once we specify the necessary routes, we'll build the controller that handles the various routes. The `SessionsController` will consist of `new`, `create`, and `destroy` actions, which correspond to creation of login form, submission of login form, and logging-out, respectively. The `new` action and it's associated view template (`new.html.erb`) may look somethiing like this:

```ruby Ex2: 'new' Action Inside SessionsController
# Inside SessionsController
def new
end
```

```erb Ex3: 'new.html.erb' - Form for Login
<div class='well'>
  <%= form_tag '/login' do %>
    <div class='control-group'>
      <%= label_tag :username %>
      <%= text_field_tag :username %>
    </div>
    <div class='control-group'>
      <%= label_tag :password %>
      <%= password_field_tag :password %>
    </div>
    <%= submit_tag "Login", class: 'btn btn-success' %>
  <% end %>
</div>
```

In `Ex:2`, we have an empty `new` action which simply renders the `new.html.erb` template by default. Inside our template, we have a non model-backed form - it is NOT model-backed because `sessions` is not an ActiveRecord object, meaning we don't have a `sessions` table in our database. The `form_tag '/login'` indicates that the the form will be submitted to `/login` with an HTTP method of `post`, which of course maps to `sessions#create`. The login form will look something like this:

{% img center /images/login_form.png 350 350 'login_form' %} 

Now, we will handle the submission of the form inside `sessions#create`. This code is reflected below:

```ruby Ex4: 'create' Action Inside SessionsController
def create
  user = User.find_by(username: params[:username])

  if user && user.authenticate(params[:password])
    flash[:notice] = "You successfully logged in."
    session[:user_id] = user.id
    redirect_to root_path
  else
    flash[:error] = "There's something wrong with your username or password."
    redirect_to login_path
  end
end
```

When we submit the form, we gain access to the `params` hash that will look something like this: `{:username=>'some username', :password=>'some password'}`. In line 2, we check the database to see whether or not a `user` object with the specified username exists. Line 4 states that, if such user exists and if the submitted password matches the password stored in the database (ie. password that was stored when a user first registers, which is not shown in this blog post but accomplished via a model-backed user form with `username` and `password` fields), the authentication is sucessful. Line 6 is critical because the `session` tracks the logged-in user. It's also important that we NEVER save objects into a session because sessions are cookie-backed and cookies have a relatively meager 4KB size limit. This is why we're storing the `user_id` instead of the entire `user` object.  

Using `session[:user_id]`, we can selectively display UI elements and hide certain elements that unauthenticated users shouldn't have access to. To facilitate this process, we'll introduce several methods in our `ApplicationController`. 

```ruby Ex5: Some Methods Inside ApplicationController
class ApplicationController < ActionController::Base
  helper_method :current_user, :logged_in?

  def current_user
    @current_user ||= User.find(session[:user_id]) if session[:user_id]
  end

  def logged_in?
    !!current_user
  end

  def require_user
    if !logged_in?
      flash[:error] = "Must be logged in to do that."
      redirect_to root_pth
    end
  end
end
```

The `current_user` method checks to see whether or not `session[:user_id]` exists, and then returns the user object corresponding to that `user_id` if the session does indeed exist. If it doesn't, it will return `nil`. The `logged_in?` method converts what was returned from `current_user` into a boolean value - a `user` object into `true`, and `nil` into `false`. If we seek to suppress certain features in the UI we can use a conditional to display certain elements. For example, the link to create a new post should only be accessible when a user is logged in. This can be accomplished very easily inside the view template:

```erb Ex6: Suppressing UI Elements in View Template
<% if logged_in? %>
  <%= link_to 'New Post', new_post_path, class: 'btn btn-success' %>
<% end %>
```    

Here we are calling the `logged_in?` method and displaying the `New Post` button only when the user is logged in. Notice that we have access to this method inside the template even though the method was defined inside `ApplicationController`. This is because we included `helper_method :current_user, :logged_in?` inside `ApplicationController`, which essentially copies the specified methods and moves them over to `ApplicationHelper`.  

Up to this point, we have effectively hidden certain UI elements in our view templates to prevent unauthenticated users from accessing these elements. However, a user can still navigate to a URL such as `/posts/new`, which will then display a new post form, regardless of whether a user is logged in or not. To prevent this, we will call the `require_user` method inside all controller actions that should be prohibited from users that are logged-out. For example, we wouldn't want an unauthenticated user from creating, editing or updating a post, so we will call the `require_user` method in the `new`, `create`, `edit`, `update` actions inside our `PostsController`. This method will flash an error and prevent non-logged in users from accessing certain features.

To summarize, whenever we want to prohibit non-logged in users from performing certain actions, we have to suppress BOTH the UI element AND the route. We suppress the UI element by using an `if logged_in?` inside a view template, and we suppress the route by calling `require_user` inside a controller action. 

####Simulating the Logout

Simulating the logout is very easy. As we can see from our custom route, `get '/logout', to: 'sessions#destroy'`, logging out is handled by the `destroy` action inside `SessionsController`. 

```ruby Ex7: 'destroy' Action Inside SessionsControllers
def destroy
  session[:user_id] = nil
  flash[:notice] = "You've logged out"
  redirect_to root_path
end
```

Here we simply set the `session[:user_id]` to `nil`. Now, whenever we call the `logged_in?` method from any of our view templates (like in `Ex6`), the UI element will not be displayed because the `current_user` method will return `nil`, which is then be converted to `false` by the `logged_in?` method.  

####Summary

In this post, we walked through the authentication system from scratch. We introduced the session to keep track of which user is logged in, and using this information, we were able to suppress certain features that should be unaccessible to unauthenticated users. 




















