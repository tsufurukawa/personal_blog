---
layout: post
title: "Deployment Pipeline"
date: 2014-07-25 09:40:40 -0700
comments: true
categories: [Deployment Pipeline, Heroku, Rails]
keywords: Ruby on Rails, Tealeaf Academy, deployment pipeline, paratrooper, heroku, staging
description: Setting up simple deployment pipeline using paratrooper gem and a staging server
---

####Background
_Deployment pipeline_ is the process of breaking up automated deployment steps to ensure bugs are detected and broken features are addressed before an application is set to production. A web application typically runs on three environments: _development environment_ running on a developer's local machine, _test environment_ for running tests, and _production environment_ where real users interact with the application. The underlying problem with this setup is that there's no guarantee code that works flawlessly in local development environment will work the same when deployed to production. That is, development and production environments are inherently different with different configurations, so deploying straight to production from development is a risky process.

The solution is to have a _staging environment_ that __mimics the production environment__ as closely as possible. Essentially, we want to create a platform where we can check our code in a production-like setting so that any new feature can be thoroughly tested and bugs can be accounted for before having it affect our actual users. Imagine deploying thought-to-be-working code to production and later discovering broken features in the application - you'll likely have to shut down the site, thereby disrupting the end user experience. A staging environment can guard against such catastrophe.

####Deployment Pipeline at Tealeaf Academy
Deployment pipelines can become very complex, especially for larger projects. For the application we are currently working on at Tealeaf, we adopted a simple but effective deployment pipeline that can be summarized as follows:

1. Follow the Github Flow process - after you complete a feature on a branch, merge the pull request back to the master branch.  
2. Run entire test suite - _don't break the build_.  
3. Deploy code to staging - test on staging server manually to ensure that the new feature works.  
4. Deploy code to production - remember the staging server should be as close to the production server as possible.

####Setting Up the Deployment Pipeline
Now, we'll walk through how to set up the deployment pipeline with multiple environments on Heroku. We will follow this [Heroku tutorial](https://devcenter.heroku.com/articles/multiple-environments) to first set up the staging environment, and then use the [paratrooper gem](https://github.com/mattpolito/paratrooper) to simplify the deployment process with quick/concise rake tasks.

######1. Fork an Existing Application
Let's assume we already have an existing application running in production, in which case, we can duplicate the staging application via _forking_.

```
heroku fork -a production_app_name staging_app_name
git remote add staging git@heroku.com:staging_app_name.git
```

The great thing about forking an application is that it copies over the config vars, re-provisions all add-ons, and copies all Heroku Postgres data too, so that we don't have to do these manually. In the second line, we are adding a new `staging` remote repository.

######2. Simplify Heroku Deploy with Paratrooper Gem
The _Paratrooper_ gem allows us to write a set of rake tasks that can simplify the deployment process.

First, add this line to the Gemfile:

```ruby 
gem 'paratrooper'
```

Although we can customize the method however we want, the Paratrooper wiki provides a _Sensible Default Deployment_ code:

```ruby
require 'paratrooper'

  namespace :deploy do
    desc 'Deploy app in staging environment'
    task :staging do
      deployment = Paratrooper::Deploy.new("staging_app_name", tag: 'staging')

      deployment.deploy
    end

    desc 'Deploy app in production environment'
    task :production do
      deployment = Paratrooper::Deploy.new("production_app_name") do |deploy|
        deploy.tag              = 'production'
        deploy.match_tag        = 'staging'
      end

      deployment.deploy
    end
  end
```

The above code is a rake task - put it under __lib/tasks__ directory as __deploy.rake__ and tweak it so that it uses your own Heroku app name. The code performs the following tasks:

*  Create or update a git tag (if provided)
*  Push changes to Heroku
*  Run database migrations if any have been added to db/migrate
*  Restart the application
*  Warm application instance

Essentially Paratrooper takes all the steps necessary for deployment and wraps them in a single rake command - `rake deploy:staging` to deploy to staging, and `rake deploy:production` to deploy to production.

######3. Modify Config Vars on Heroku and Create Staging Config File
Notice now that if you inspect the `RAILS_ENV` and `RACK_ENV` environment variables for our staging app, they both say "production". Thus by default it will load the configuration file under __config/environments/production.rb__. This may be okay if the production and staging servers have identical configurations; however, typically we should create a separate __staging.rb__ file, in which we specify the exact settings we want for the staging environment (such as turning on asset pipeline or not, sending out email or not, or charging credit card for real or not, etc).

We can modify the environment variables as follows:

```
heroku config:set RAILS_ENV=staging --remote staging
heroku config:set RACK_ENV=staging --remote staging
``` 

Now, whenever we deploy to staging, Heroku will run the staging configuration file (__staging.rb__).

####Summary
In this post, we talked about the importance of having a staging server and how to set up a simple deployment pipeline. Again it is very important for the __staging server to be as close to the production server as possible__, or else it defeats the whole point of having a staging server.