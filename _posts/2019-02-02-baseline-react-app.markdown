---
layout: post
title:  "React/Rails/Redux/React Router/Heroku: Here's a Baseline for Building Out a React App for Deployment to Heroku"
date:   2019-02-2 12:00:00 -0400
categories: coding
---

A quick post here. Since building [Browseum](/projects/#browseum) and deploying it to Heroku, I've been wanting to play with React more. I've also been thinking how it would be nice to have a clone-able, baseline app for future React projects.  This baseline app would have the "fixin's" already built in (like a Rails API, Redux state, and React Router) and would have the foundation necessary for deployment to Heroku, so basic [Heroku troubleshooting](/coding/2018/11/02/notes-troubleshooting-heroku.html) could be avoided. That way, when someone wanted to play around with React, she or he could clone the app and get building fast. Connecting a React App to a Rails API, Redux state, etc. is a great learning experience, of course, but sometimes it is useful to have a springboard already there to dive in quickly.

So here it is: a [baseline React app](/projects/#baseline-react-app) with the fixin's built in. Please feel free to clone the repository and play around. The instructions for getting the app up and running are in the repository's README, and they describe how to use the test components or the browser console to check that the Rails API, Redux, etc., are all working together.  I re-tested the instructions yesterday with a clone of the baseline app, and everything went fine, including deploying the baseline app to Heroku. A copy of the README is below in case you'd like to get a preview of what's involved. Hope you find the baseline app helpful!

-----

# README

This is a baseline I set up for a React app with a Redux state, React Router, and a Rails API, intended for deployment to Heroku. The intention is to use clones of this as a launching pad for future React apps. In other words, this is a barebones working app with a "test" React component, Redux state, Router, Rails controller, and Rails model -- all of which are there simply to show that everything is all hooked up and working together. Hope this it saves you some time if you are looking for a React/Rails playground for kicking around ideas.  The instructions below are intended to be checks that you can run to make sure everything is working properly before you flush out your app.  

I found particularly helpful advice and guidance at these sites:
  - https://www.fullstackreact.com/articles/how-to-get-create-react-app-to-work-with-your-rails-api/
  - https://blog.heroku.com/a-rock-solid-modern-web-stack

Please note:

  - The app assumes you are working from Ruby version 2.5.3.  To work from another version, you will have to update the Gemfile, but please make sure your version of Ruby is compatible with Heroku first, if you plan to deploy there.

  - The app also assumes you have basic command line tools in place like bundle, npm, and Heroku.

  - The app does not implement Devise or ActiveAdmin.

To get this baseline app up and running and check that things are working properly:

1) Fork and clone the repository.

2) From within the root directory, don't forget to run `git init .` to begin tracking changes with git.

3) Get the database up.  Nothing's moving forward without it.

  - Because the app is set up to be compatible with Heroku, it requires a Postgres database.  Therefore, before launching the development server locally, please make sure you have downloaded the Postgres stand-alone app: https://postgresapp.com/. Then click on the icon make sure the Postgres app is running.

  - You'll probably want to change the names of the development and test databases (this is not necessary for production, as Heroku takes over at that point).  To change the names of the databases, head to config/database.yml.  Under `development:` and `test:`, find the `database:` keys, and there you can rename your development and test databases, instead of `precedent_development` and `precedent_test` respectively.

  - Then in your terminal, from the root directory, run `rake db:setup`.  In the Postgress app, you should see your databases listed.  Running `rake db:setup` will also migrate a Rails migration to initialize `TestModel` and will seed some TestModels with simple messages.  This is all to make sure later that the database and ActiveRecord are all synced up.  

4) Get the Ruby libraries and Rails dependencies you'll need.

  - From the root directory, run `bundle install`.

5) Get the React dependencies you'll need.

  - `cd` into the '/client' directory.
  - From there, run `npm install`.
  - `cd` back into the root directory.

6) Check that everything is working.

  - From the root directory, run `rake start`.  This will launch both the React front end and Rails API back end.
  - Head to localhost:3000.
  - You should see something very simple on the home page: two links and a button that lets you "add 1s".  
  - If you click the links and they work, React Router is up and running.
  - If you click the button and more "1s" appear, Redux is up and running.
  - To test if the Rails API is up and running, a fetch request is made to the Rails API every time the home page loads. To get confirmation that things are working ok with the Rails API, ActiveRecord, etc., make sure you are on the home page.  Open up your developer console.  Hit refresh.  If everything is ok, you should see a log in the console that says:

    `"You made it to the Rails API Test Controller and the database is connected.  TestModel #1 says: 'You pulled a model from the DB - I'm Number 1''"`


7) You might want to check things are working ok with Heroku. To do so, stop your local server and run the following commands from your root directory:

  - `heroku apps:create`
  - `heroku buildpacks:add heroku/nodejs --index 1`
  - `heroku buildpacks:add heroku/ruby --index 2`
  - `git add .`
  - `git commit -m "[Your commit message here]"`
  - `git push heroku master`
  - `heroku open`

If you see the app on the Heroku site, then everything is working ok.

Please note: running `git push heroku master` should initialize the database and seed it, but if you are having trouble getting the database message in Step 6 above, try running `heroku run rake db:seed`, and then open the Heroku page again.

9) Remove what you don't want.

There are a number of components, reducers, routes, etc. labeled "test"-something. Feel free to keep them to make sure everything is hooked up, but you'll probably want to delete them at some point.  These "test" things appear in

- app/controllers -> test_controller.rb
- app/models -> test_model.rb
- config/routes.rb -> see generally
- client/src/reducers -> see generally
- client/src/actions -> testActions.js
- client/src/components_container -> testComponent.js
- client/src/components_presentational -> aboutTestComponent.js
- client/src/App.js -> see generally
- db/migrate -> see generally
