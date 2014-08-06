---
layout: post
title: "Authentication - Part I"
date: 2014-05-22 20:15:01 -0700
comments: true
categories: [Rails, Authentication]
keywords: Ruby on Rails, Tealeaf Academy, authentication
description: How to implement authentication in Rails using bcrypt-ruby and has_secure_password
---

####How Password Authentication Works

The theory behind how authentication works in Rails is pretty straightforward. We save passwords through a *1-way hash*, meaning that we hash some string (ie. when user first registers) into a long undecipherable hash token. We then perform authentication, not by decrypting this hash token, but rather by hashing another string (ie. when user logs in) and then checking if the two hashes match. 

It's important that we never store the actual password string (like "password") in our database - we instead want to store a string representation of the password in the form of an undecipherable token. This is also called a *1-way hash digest*. When we authenticate, we compare the two digests, NOT the actual password string. In fact since the hashes are "1-way", we cannot convert the digest back into the password string. 

####How to Implement Authentication in Rails

######1. Create `password_digest` Column in Database

First, we need a column to keep track of the long token. This column must be called `password_digest`. 

{% img left /images/password_digest.png 350 350 'users_table' %}

This figure shows how a `users` table may be represented inside of a database. The `password_digest` attribute stores the long token or digest. The way we generate this token is through a gem called `bcrypt-ruby`, which uses a complicated hashing algorithm to convert a chunk of data (eg. your user's password) into an undecipherable string. Because this process is not reversible it's impossible to go from the hash back to the password.  

######2. `has_secure_password` in Model
At the model layer, we use the `has_secure_password` method to save the passwords in our database. `has_secure_password` uses the bcrypt gem to generate the one way hash. 

```ruby User Model with has_secure_password
class User < ActiveRecord::Base
  has_secure_password validations: false
  validates :password, presence: true, on: :create, length: { minimum: 5 }
end
```

`has_secure_password` has built in validations, which may or may not be useful. In the code above, we remove the default validations by delcaring `validations: false`. We then declare our own validations manually (although this step is not necessary, it's recommended because it gives us better control of which validations to include and allows us to be more explicit in specifying the validations).

######3. Authenticate Using `.password` and `.authenticate`

`has_secure_password` gives us a couple methods that allow us to complete the authentication process. The first is a `.password` virtual attribute setter method. By calling `.password` on a `user` object, `bcrypt` automatically hashes the password string, which is then stored under the `password_digest` column. It's noteworthy that there is no getter method for `.password`, which makes intuitive sense because the original password strings are unrecoverable. 

The second method that `has_secure_password` give us is `.authenticate`. This method is similar to `.password` in that we pass in a string which automatically gets converted into a hash token. `.authenticate` will compare the submitted digest with the stored digest in our database. If the hash tokens match, it'll return the user object which evaluates to true; otherwise, it'll return false. 

The following code shows these steps in action:

```ruby
u = User.new('bob')
u.password = 'password' # using the '.password' setter method - sets 'password_digest'
u.save # saves user object in database

user = User.find_by(username: 'bob') # looks for a 'bob' user object from database
user.authenticate('some password') # returns false because the hash digests do not match
user.authenticate('password') # returns the 'bob' user object because the hash digests do match
```

####Summary

In this post, we talked about the theory of authentication and how Rails implements them in the database and model layers. In the next post, we'll use a practical example to see how this all comes together. 
















