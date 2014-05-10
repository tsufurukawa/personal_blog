---
layout: post
title: "Part I: Rails Conventions"
date: 2014-05-08 22:33:15 -0700
comments: true
categories: [Rails, Rails Convention]
---

####Some Basic Rails Conventions

Now, let's talk Rails. Rails is...interesting, to say the least. It's only been a week since I've been exposed to the core ideology behind Rails, so I am far from knowing the ins and outs of the language. Rails is centered around the paradigm of *Convention Over Configuration*, which in layman's terms - follow the convention and you'll be grateful, and you're screwed if you don't. I think these conventions are what makes everything in Rails seem so magical, while simultaneously confusing. Until you wrap your head around the fact that a lot goes behind the scenes and some things you just have to take for granted, learning Rails could be quite a challenge. 

Here, I present some crazy conventions that I encountered and initially had difficult time digesting; thus, I deemed important to take note. 

In Rails (or any MVC framework for that matter), the *View* contains the content that will be rendered and returned to the browser as part of the HTTP response. An example erb template and the controller that renders the template, are shown below: 

``` erb Example ERB Template Under app/views/posts/index.html.erb
<%= @posts.each do |post| %>
  <h4>Title: <%= post.title %> </h4>
  <p>Description: <%= post.description %></p>
<% end %>
```

``` ruby Posts Controller that Renders the View Template Above
class PostsController < ApplicationController
  def index
    @posts = Post.all
  end
end
```

Here the `PostsController` is extracting all `Post` instances stored in the database called `posts` (named in such way by convention) and storing it in an instance variable called `@posts` allowing the erb template to access the data without directly communicating with the Model / Database layer. The template can then iterate through the `post` objects and display the data contained within those objects (which are essentially table rows when speaking on the database layer). So far, so good. 

Every controller action maps to a respective view template (unless of course we redirect instead of render), so as an application grows in complexity, we can expect for code to get cluttered and we often find ourselves repeating the same code in multiple files. It turns out, we can create special templates called *partial*, in which we can embed the repetitive code. We can then *pull in* the partial from multiple files whenever we seek to access its content. A code is worth a million words, so here is an example:

``` erb Pulling In a Partial Inside app/views/posts/index.html.erb  
<p>This is my main content template, and I'm going to pull in the partial on the next line.</p>

<!-- Pull in the partial -->
<%= render 'shared/content_title', title: 'All Posts' %>

<p>This paragraph will be rendered after the partial content</p> 
```

``` erb The Partial That's Being Called Inside app/views/shared/_content_title.html.erb
<h4>This is a Partial</h4>
<p><%= title %></p>
```

All partial files are named with a preceding underscore (again by convention), but strangely enough whenever we render the partial, it is called without the underscore, as shown by the top code. We are also passing a local variable `title` into the partial, which allows us to render code across multiple files that is similar in structure, but varies in content. Specifically, we can call this partial from any view file (such as show.html.erb or edit.html.erb), and by passing in different values for `title` we can render dynamic content while maintaining the structure of having the single `h4` and `p` tags.

####To Be Continued

Since this post is getting WAAAY longer than I anticipated, I will continue this discussion on partials and how Rails convention can get a little convoluted on the next post. 



