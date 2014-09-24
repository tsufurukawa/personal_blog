---
layout: post
title: "Action Mailer and Intercepting Emails"
date: 2014-07-26 17:55:37 -0700
comments: true
categories: [Email, Rails]
keywords: Ruby on Rails, Tealeaf Academy, staging, ActionMailer, interceptor
description: Using Action Mailer interceptors for sending emails in staging
---

In the previous post, we discussed how to set up a staging environment that mimics the production environment. One of the few configuration differences that we'll want to address is the way in which we handle email submission. For example, say we have an application that sends out a welcome email after a user signs up for an account. Of course in production, we want the email to flow straight to the user's inbox; however in staging, we want to prevent spamming our users and instead want the test emails to flow to a designated person - say an admin. Fortunately in Rails, Action Mailer provides hooks to intercept every mail, which then allows us to make modifications to messages right before they are handed over to the delivery agents.

####Email Sending with Action Mailer
Going back to the example described above, we send out a welcome email after a user registers. The method that accomplishes this is defined inside __app/mailers/app_mailer.rb__:

```ruby Ex1: Welcome Email Method
class AppMailer < ActionMailer::Base
  def send_welcome_email(user)
    @user = user
    mail from: 'info@app.com', to: user.email, subject: "Welcome to this Awesome App!!"
  end
end
```

Note that this method does not actually send out the email - it simply returns a `Mail::Message` object which can then be delivered by calling the `deliver` method. This process is shown inside the `UsersController#create` action where a new user object is created and an email is sent to the said user.


```ruby Ex2: Sending Email Upon Registration
def create
  @user = User.new(params.requie(:user).permit(:name, :email, :password))
  if @user.save
    AppMailer.send_welcome_email(@user).deliver
    flash[:success] = "Welcome!! You've successfully registered!!."
    redirect_to home_path
  else
    render :new
  end
end
```

Now if we use a third party email service provider such as [Mailgun](http://www.mailgun.com/), the configuration inside __config/environments/staging.rb__ and __config/environments/production.rb__ will look something like:

```ruby Ex3: Email Sending Configuraiton for Staging and Production
ActionMailer::Base.smtp_settings = {
  :port           => ENV['MAILGUN_SMTP_PORT'],
  :address        => ENV['MAILGUN_SMTP_SERVER'],
  :user_name      => ENV['MAILGUN_SMTP_LOGIN'],
  :password       => ENV['MAILGUN_SMTP_PASSWORD'],
  :domain         => 'my_app.herokuapp.com',
  :authentication => :plain,
}
ActionMailer::Base.delivery_method = :smtp
```

Now if we deploy our app, the emails will be delivered to the user's inbox. The problem however remains - for staging, we want the email to be delivered to a default user (i.e. the admin) instead of to the newly registered user.

####Intercepting Emails
The implementation is actually rather simple. First, we create an _interceptor_, which allows us to edit an email before it's delivered. The file is called __staging_email_interceptor.rb__ and should be placed under the __lib__ directory.

```ruby Ex4: Staging Email Interceptor
class StagingEmailInterceptor 
  def self.delivering_email(message)
    message.to = ENV["STAGING_EMAIL_RECIPIENT"]
  end
end
```

`delivering_email` is a class method that gets triggered right before the email is sent, and `message` refers to the `Mail::Message` instance that is returned from the `send_welcome_email` method we defined previously. So essentially after `send_welcome_email` is called, the message object gets sent to `delivering_email` where the `to` parameter is overriden to the `STAGING_EMAIL_RECIPIENT`, which is an environment variable pointing at the admin's email address to where all staging emails will be filtered.

Lastly, we need to register the interceptor with the Action Mailer framework. We can do this in an initializer file __config/initializers/register_email_interceptor.rb__. 

```ruby Ex5: Register the Interceptor
ActionMailer::Base.register_interceptor(StagingEmailInterceptor) if Rails.env.staging?
```

The conditional at the end is important because we only want the inteceptor to get triggered if the `RAILS_ENV` environment variable is set to `staging`. Thus, in `production` the interception will be bypassed and the email will be sent to the registered user's address.

####Summary
In this post, we discussed how to send an email using the Action Mailer framework. Then we used interceptors to filter all test emails in staging to the admin user's inbox.

##Aside
I haven't posted any quotes recently, so here's one from my idol growing up.

> “Everything negative – pressure, challenges – is all an opportunity for me to rise.” 
> <cite><sub> - Kobe Bryant</sub></cite>

