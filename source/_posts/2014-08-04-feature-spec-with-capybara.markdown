---
layout: post
title: "Feature Spec with Capybara"
date: 2014-06-22 21:19:29 -0700
comments: true
categories: [RSpec, Capybara, Rails, Testing]
keywords: Ruby on Rails, Tealeaf Academy, Capybara, RSpec, testing, feature spec
description: Simple feature spec using Capybara with RSpec
---

When designing a web application, or any computer software for that matter, having automated tests is absolutely critical. In general, tests can be broken down into 3 categories:

__1. Unit Tests__ - testing different components in isolation.   
__2. Functional Tests__ - testing multiple components in collaboration and making sure individual components work well with each other.  
__3. Integration Tests__ - following a business process and ensuring the components can work together to achieve a business objective.

Now, in the context of Rails, these sections can be broken down as:  
1. Models, Views, Helpers, Routes -> Unit Tests  
2. Controllers -> Functional Tests  
3. Emulating End User "Browser Experience" -> Integration Tests  

In this blog post, we'll go through a simple feature spec using  __Capybara__ with __RSpec__.

####Basic Setup
Capybara helps us test web applications by simulating how a real user would interact with our app. It works seamlessly with RSpec, and most importantly, it's got an intuitive API that reads very cleanly. Let's look at an example.

Start off by installing the `capybara` and `capybara-email` gems inside your Gemfile.

```ruby Ex1: Install Capybara in Gemfile
group :test do
  gem 'capybara'
  gem 'capybara-email'
end
```

Next, inside `rails_helper.rb`, add these lines of code:

```ruby Ex2: Inside Test Helper File
require 'capybara/rails'
require 'capybara/email/rspec'
```

Lastly, create a file called `user_resets_password_spec.rb` inside the `spec/features` directory. We're now ready to start writing out the test!!!

As the name of the file suggests, this particular feature spec mimics the user experience of resetting his/her password. The basic setup of this feature spec is as follows:

```ruby Ex3: Setting Up "Reset Password" Spec
require 'rails_helper'

feature "User resets password" do
  scenario "user forgets and resets password successfully" do
    # Setup the data
    user = Fabricate(:user, name: "Joe Smith", email: "test@example.com", password: "password")
    clear_email
  end
end
```

Here, we're using an object generation library called `Fabrication` to generate the `user` object. The `clear_email` method is a convenient method given to us by the `capybara-email` gem, which simply clears the email message queue. (Note: Sending emails is a huge topic in and of itself, but basically whenever emails are sent using ActionMailer, they get stored in a message queue. We must clear this queue to ensure a clean slate and that any extraneous emails are wiped out.)

####Simulating the User Experience
When a user forgets the password, the first thing he/she will do is to click on the "Forgot Password?" link. The sign-up for this particular app looks something like this, and the user experience can be mimicked with the following bit of code:

{% img center /images/sign_up_form.png 400 350 'sign_up_form' %} 

```ruby Ex3: User Clicks on "Forgot Password" Link
visit sign_in_path
click_link "Forgot Password?" 
```

Notice how readable this code is!! This is the beauty of working with the Capybara API - even a non-programmer can read and decipher the code.

Once the user clicks on the link, it will redirect to a page where the user is prompted to enter an email address. Then inside our server side code (specifically, inside the `ForgotPasswordsController`, we send out an email with directions to reset the password if the said email address does indeed exist in our database. 

{% img center /images/forgot_password_form.png 600 260 'forgot_password_form' %}

The Capybara code that mimics this user experience is again very straightforward. 

```ruby Ex4: User Fills out Email Address
fill_in "Email Address", with: user.email
click_button "Send Email"
```

Upon submission of the form, the user will receive an email with a `"Reset Password"` link. Clicking on this link will redirect the user to the "Password Reset" page where the user is prompted to enter a new password. This sequence of events is depicted with the following lines of code:

```ruby Ex5: User Clicks on "Reset Password" Link and Enters New Password
open_email(user.email)
current_email.click_link "Reset Password"

fill_in "New Password", with: "new password"
click_button "Reset Password"
```

{% img center /images/reset_password_form.png 550 260 'reset_password_form' %}


The `open_email` method finds the latest email sent to the specified address (`user.email`), and sets the `current_email` so that we can inspect its content. 

Finally, the user will fill out the sign-in form with the `"new password"` to ensure he can successfully log in.

```ruby Ex6: User Signs In with New Password
fill_in "Email Address", with: user.email
fill_in "Password", with: "new password"
click_button "Sign in"
expect(page).to have_content("Welcome #{user.name}!! You successfully signed in.")
clear_emails
```

The `expect(page).to have_content(...)` is an _assertion_, which is basically a way of verifying if the results are what we expect.

The complete spec is as follows:

```ruby Ex7: The Complete Feature Spec for Resetting Password
require 'rails_helper'

feature "User resets password" do
  scenario "user forgets and resets password" do
    user = Fabricate(:user, name: "Joe Smith", email: "test@example.com", password: "password")
    clear_emails

    visit sign_in_path
    click_link "Forgot Password?" 

    fill_in "Email Address", with: user.email
    click_button "Send Email"

    open_email(user.email)
    current_email.click_link "Reset Password"

    fill_in "New Password", with: "new password"
    click_button "Reset Password"

    fill_in "Email Address", with: user.email
    fill_in "Password", with: "new password"
    click_button "Sign in"
    expect(page).to have_content("Welcome #{user.name}!! You successfully signed in.")
  
    clear_emails
  end
end
```

####Summary
As we saw from this example, Capybara has an extremely intuitive set of APIs that makes integration testing simple and even enjoyable (at times). Having automated tests helps guard against _regression_, a common problem in software development where an application stops functioning as intended after addition of a new feature. Automated testing can become quite tedious, especially during early stages of application development; however, without it, it is simply impossible to tell whether or not the entire system is bug-free after a new feature is introduced. 