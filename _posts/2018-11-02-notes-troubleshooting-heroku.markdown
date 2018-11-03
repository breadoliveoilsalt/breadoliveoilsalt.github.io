---
layout: post
title:  "Rails/React: Notes on Troubleshooting Heroku Deployment for a React App with a Rails API"
date:   2018-11-02 12:00:00 -0400
categories: coding
---

One of the apps I've been working on, Browseum, has a React front-end and a Rails API back-end. For apps with this architecture, some people create two project folders - one for the React code, and one for the Rails code. When starting to work on Browseum, I decided to go a different route and combine the front-end and back-end into one project folder.  A great guide on how to do this is [here](https://www.fullstackreact.com/articles/how-to-get-create-react-app-to-work-with-your-rails-api/).

Browseum is in good enough shape that I wanted to deploy it to [Heroku](https://www.herku.com/).  When I started researching how do this, I quickly became nervous.  There are countless accounts of people struggling to deploy a React app with a Rails API and regretting not breaking their project into two folders (which, it seems, would then require setting up two Heroku dynos, one for each folder).

Fortunately, on the Heroku blog itself, there is a [great, fairly-recent guide](https://blog.heroku.com/a-rock-solid-modern-web-stack) on the building blocks needed to deploy a React app with a Rails API.  I took the lessons from the guide and did my best to back them into the substantially-completed Browseum.  

I can't state enough how helpful the guide was, and I'm very grateful to the [author](https://blog.heroku.com/authors/charlie-gleason) for all the effort he put into it. As with all things, deploying Browseum to Heroku still presented problems to troubleshoot, and I thought I would describe here what I encountered in case anyone finds herself or himself in a similar situation.

One thing to note before diving in: the Heroku guide goes into how to get the app to work with Devise and ActiveAdmin.  I did not follow any of those steps for Browseum, and ultimately the app still deployed fine. In other words, it's not essential to follow the guide's steps on Devise and ActiveAdmin if accounts and passwords are not important to your app, so skipping those parts may save you some time.  

*****
<p/>

<h4 class="text-centered bold underlined"> 1. Changing the database from SQLite3 to Postgres </h4>

Heroku does not support SQLite3, and apps that are deployed to Heroku must rely on a Postgres database. There actually wasn't much to troubleshoot here, but this fact about Heroku took me by surprise, so I thought I would flag it.  I wish I had been building my app from the start with Postgres, just to avoid any major problems.  Fortunately, following the Heroku guide on this point was pretty seamless (see Step 2 of the guide), and my migrations still worked.  

*****
<p/>

<h4 class="text-centered bold underlined"> 2. Upgrading Ruby Versions </h4>

I use [RVM](https://rvm.io/rvm/basics) to manage my Ruby versions.  For historical reasons, I had been running Ruby version 2.3.1 in my terminal, and so that's what I used when building Browseum.  This version of Ruby was documented in my Gemfile.  Heroku does not support v2.3.1, however, so I had to upgrade my Ruby version, in both my terminal shell and the app. The Heroku CLI seemed to be hyper aware of when I wasn't using an approved Ruby version in my shell, as well as when my app still contained some aspect of a non-approved Ruby version, throwing errors constantly.

Because I initially moved too fast and didn't read the fine print in the [Heroku docs](https://devcenter.heroku.com/articles/ruby-support#ruby-versions), I ended up upgrading my Ruby version twice: the first time in a poor, inefficient way that installed an unsupported Ruby version, and the second time in a better, more efficient way that got the job done.  

Here's the better way, using RVM:

1. Assuming you are working on a Mac, using a text editor, check your `.bash_profile` and `.bashrc` files to make sure that the RVM path language is the last thing to run in these files. The files will be located in your computer's root directory, but they might be hidden when viewing your root directory in a Finder window (hit `command-shift-.` to make the hidden files appear).  Assuming by now you have installed the heroku CLI, this installation may have inserted new language at the end of these files, causing RVM to throw errors, because RVM is very picky about its desire for RVM language to be last in the files.  Seeing this warning in your terminal when you try to run a RVM command could be a clue that `.bash_profile` and `.bashrc` need to be adjusted:

    >Warning! PATH is not properly set up, /Users/adistinti/.rvm/gems/ruby-2.3.1/bin is not at first place.

    Simply copy and paste the RVM language in each of these files, moving the language to the very bottom.

2. Install a [version of Ruby supported by Heroku-18](https://devcenter.heroku.com/articles/ruby-support#ruby-versions) via the terminal.  I went with v2.5.3.

    ```
    rvm install 2.5.3
    ```

3. Switch to that version of Ruby in your terminal (this command will make it the default version for all your terminal shell windows going forward):

    ```
    rvm --default use 2.5.3
    ```

    And make sure to update your Gemfile to reflect this Ruby version.

    ```
    ruby '2.5.3'
    ```

4. RVM keeps separate gem files for each version of Ruby.  In other words, the gems I had installed for v2.3.1 were no longer available.  To keep things simple and efficient, in your terminal, re-install Bundler:

    ```
    gem install bundler
    ```

    Then, to install only the gems relevant to your app run:

    ```
    bundle install
    ```
    My app worked fine after this.  If you are having trouble, however, try the command:

    ```
    gem pristine --all
    ```

***NOTE***: Prior to the steps above, I ran in circles accidentally installing an unsupported version of Ruby and then copying every gem I had to this version, which took forever.  I won't go into the gory details.  But it is worth nothing that if you don't follow the steps above and if you see the following error in your terminal log after installing a Ruby version and installing gems...

> It seems your ruby installation is missing psych (for YAML output).
To eliminate this warning, please install libyaml and reinstall your ruby.

...then here's what you should consider doing, based on [this thread](https://github.com/Homebrew/homebrew-core/issues/11636).  Reinstalling libyaml was no help. Instead, this did the trick:

1. Find your hidden `.rvm` folder in your computer's root directory, locate the new ruby version you just installed, and completely delete (or move to the trash) the two folders relating to the psych gem.  For me these two folders were:

    ```
    User/[username]/.rvm/gems/ruby-2.3.8/gems/psych-3.0.2
    User/[username]/.rvm/gems/ruby-2.3.8/extensions/x86_64-darwin-17/2.3.0/psych-3.0.2
    ```

2. Then run:

    ```
    gem pristine --all
    ```

*****
<p/>

<h4 class="text-centered bold underlined"> 3. A Possible Solution if the App Seemingly Deploys to Heroku, but on the App's Site, You Keep Seeing "The Page You Were Looking For Doesn't Exist"</h4>

This problem drove me nuts for a while, but we finally got to the heart of the matter.  After wrestling with Ruby versions, I finally got the green light from Heroku ("build succeeded") when I pushed my code via the Heroku CLI.  However, when I tried to open the app's site, this appeared:

![](/assets/images/2018-11-02-notes-troubleshooting-heroku/image-1.png){:class="medium border centered"}

The first thing I had to figure out from some research was that, despite no indication to the contrary on the web page, this page was generated by *Rails*.  It's basically an auto-generated "404, page not found" warning from Rails when Rails is running in production mode. This error was particularly vexing because I expected React to be running the front-end, so it wasn't clear to me at all why a Rails-generated page was appearing.

In any event, the fact that Rails was generating this page was a red herring. I couldn't figure out what was wrong until, on a lark, I ran `heroku apps:create` in the terminal from the Browseum project folder to create a "new Browseum app" on the Heroku side.  Then I went through the various logs line by line.  That's when this caught my eye in the logs:

> Creating an optimized production build...
> Failed to compile.
> Module not found: Error: Can't resolve 'redux-devtools-extension' in '/tmp/build_13821c1f0cfd1698cf89c96814f0a749/client/src'

I wasn't entirely sure what this meant, but my best guess seemed that Heroku (or React, within the Heroku environment) was having a problem running or finding the `redux-devtools-extension`, even though this caused no problems when the app was run locally in  development mode. Some research eventually led me to [this thread](https://github.com/AymaneZizi/Weatherpro/issues/1), which suggests the solution is to add `/logOnlyInProduction` to the line in your React code that imports `redux-devtools-extension`. The specific procedure is described [here](https://github.com/zalmoxisus/redux-devtools-extension#14-using-in-production).  Thankfully this one change did the trick, and after pushing the modified code to Heroku, the app actually deployed and was running on its designated Heroku site.  In hindsight, I suspect the reasons why a *Rails* webpage appeared when there was a *React* error is that Heroku was able to deploy the Rails API fine, but the React "Module not found" error above meant that React did not load.  So when the browser requested the root path for the app, the only thing running was Rails, which did not specify or have a controller for a root path.  As a result, Rails issued a "404, page not found" notice.  
