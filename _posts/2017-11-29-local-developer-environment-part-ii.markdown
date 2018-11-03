---
layout: post
title:  "Lessons Learned Setting Up a Local Developer Environment, Part II: Homebrew, Ruby Version Controlling, and Gems"
date:   2017-11-29 12:00:00 -0400
categories: coding
---

**Part II -- Start Managing What You Install Via the Terminal: Homebrew, Ruby Version Controlling, and Gems**

[Part I](/coding/2017/10/30/local-developer-environment-part-i) of this blog post provided some background and basic steps to toward setting up a local development environment. It touched on some concerns specific to students at the Flatiron School, such as how to get around deleting the Learn IDE, but hopefully there is enough general applicability in these posts to be useful to anyone setting up a local developer environment for the first time.  Part II dives right in as a follow up to [Part I](/coding/2017/10/30/local-developer-environment-part-i), continuing to describe the snafus I encountered and lessons learned from tackling them.  Please make sure your read [Part I](/coding/2017/10/30/local-developer-environment-part-i) before reading Part II; otherwise, Part II won't make much sense.

*****

<p />

**Step 5: Start managing what you put on your computer, Part I: Install Homebrew**

In doing research, I kept coming across different ways to install programs or libraries via the terminal.  Some instructions would say “use homebrew.”  Others would say “use rbenv.” Yet others would say “use rvm.”  And I couldn’t understand how all these related to gems, which are installed via, well, the `gem` command.

***Lessons learned:***

* What I came to understand is that there are many different terminal programs that install, and keep track of, the other programs you put on your computer via the terminal. Let’s call the first group “handlers” and the second group “target-programs.”  You need handlers because they might have some magic under the hood that makes the installation of target-programs easier.  You also need them because often terminal programs do not show up on a graphical interface (like icons on your desktop).  Thus you need a handler to tell you what is there and perhaps remove a target-program.

* Turns out, you’ve probably been using a handler already in the IDE.  For example, when you type in `gem <gem_name>` to install a gem, you don’t type `gem` because the target-program is a gem.  You type `gem` because you are invoking a handler, which is called RubyGems (of [rubygems.org](https://rubygems.org/)) to install whatever `<gem_name>` is via the IDE.

***So*** which handler do you use?  The advice [here](https://stackoverflow.com/questions/32031822/what-is-the-difference-between-ruby-gems-and-brew-formulas) sounded good.  And I figure, go with whatever the target-program’s instructions tell you to use.  There may be functionality reasons why the creator of the target-program specifies a particular handler to install the target-program.

But before installing anything, you need the handlers:

* RubyGems should already be on your machine, likely put there by Xcode. (Or perhaps it came pre-installed; I’m not sure which).

* With respect to gems, if you try to install a gem at this point, you probably get an error about not having permission.  We’ll deal with that below.

* [Homebrew](https://brew.sh/) is a popular handler that you can install now, however.  The instructions to install it are pretty clear.  Go for it.


**Step 6: Start managing what you put on your computer, Part II: Get a Ruby Version Manager so You Can Have Your Own Ruby Playground**

Right now, when you try to install gems from your terminal via the terminal command `gem`, you probably get a big “permission denied” message.  Even though this is your Mac and you are king or queen of the castle here,  our Mac seems to view these new Ruby libraries as touching the "fundamental Ruby" associated with its operating system, and so it doesn’t want you to go there.  Unless, that is, you start typing commands containing `sudo`. `sudo` appears to unlock the basement door and let you play around with the buried secrets in your operating system.  But, if you do 2 minutes of research, you’ll quickly learn that playing with `sudo` is only for those who have the programming constitution of Indiana Jones.  You can do things with `sudo` that can muck up your entire Mac irreparably.  And so, the wise people say, it’s best to avoid using it.

So what do we do to install gems? Consider this: say you are a powerful 3 year old with vast wealth.  You also have a heck of a lot of energy to run around, try stuff, and occasionally break stuff.  You don’t want to do that to your home, so you decide to use your vast power and resources to build a playground far away from your home.  This is, fundamentally, what you are going to do now to install gems and other target-programs.  Don’t mess up your home (that is, your operating system and its beloved "fundamental Ruby").  Instead, create a Ruby playground completely separate from these things where you can run amuck, build, destroy, etc.

There are a few programs out there for creating such playgrounds on your computer, and they are generically called Ruby version managers. The two I considered were [RVM](https://rvm.io/) and [rbenv](https://github.com/rbenv/rbenv). So which should you use?

***A few lessons learned:***

* For now I'd recommend going with RVM.  Why? I actually used rbenv at first for a long while, and I liked it.  I chose rbenv, because I read how it is more lightweight and, as noted in [this summary](https://github.com/rbenv/rbenv/wiki/Why-rbenv%3F), I didn’t want to start messing with fundamental terminal commands like `cd`.

* Up until a week or two ago, I was all ready to publish this blog post describing rbenv.  Then came the day when, when in the middle of the Sinatra part of the curriculum, something went wrong and a coding lesson was throwing all kinds of mysterious errors that I couldn't solve.  During a screen share with a technical coach, the coach had me install and use RVM, because in general it seems more compatible with Flatiron's labs.  So for now, particuarly if you are a Flatiron Student, I'd say go with [RVM](https://rvm.io/).

* **BUT this is important:** When choosing which version of Ruby to install via RVM, make sure you **go with the version of Ruby that the IDE uses, which as of this post is Ruby v2.3.1** (in terminal, type `ruby -v` to see the current version).  Many of the Flatiron labs (or their underlying libraries) depend on this version of Ruby.  I originally used Ruby v2.4.2 (the latest version as of this publishing), and it turns out a ton of the errors I encountered along the way would have disappeared if I had used Ruby 2.3.1.

If you'd still like to install rbenv instead, immediately below are some paragraphs describing what I learned.  They are here simply because I previously drafted them for this post.  Fingers crossed they are of some help, maybe just for general background.  But feel free to install RVM, then install Ruby v2.3.1, set it up as your default, and then skip to Step 7.

***A few lessons learned about rbenv:***

Assuming you’d like to install rbenv, the instructions are under the readme [here](https://github.com/rbenv/rbenv).  Here’s how I thought about it, continuing to use the playground analogy.  In making your plans for a playground, the first thing you need to do is buy the land away from your home to give you the ability to have the playground in the first place.  The following, which basically installs rbenv, gets you the land:

* 1) Use homebrew to install rbenv by typing this in your terminal: `brew install rbenv`

* 2) Then enter `rbenv init` and follow the instruction.  This sets up rbenv to work with your terminal shell.

* 3) Close your Terminal window and open a new one so your changes take effect.

* 4) Follow the instructions under the rbenv readme to verify that rbenv is properly set up by using this rbenv-doctor script: `curl -fsSL https://github.com/rbenv/rbenv-installer/raw/master/bin/rbenv-doctor | bash`

Now that you’ve got the land, you need to build the fundamental structures to play on.  That is, you need to install a version of Ruby to play on (via rbenv and to work with rbenv).  As of this writing, the latest official version of Ruby is version 2.4.2, so that’s what I installed via rbenv.

* 5) To do similarly, at the terminal, enter `rbenv install 2.4.2`. [**Remember** this caused me trouble later in the Sinatra curriculum for Flatiron School.  I should have installed version 2.3.1 of Ruby.]

Now that you’ve got your playground land (rbenv) and some structures to play on (version 2.4.2 of Ruby, isolated away from your operating system’s Ruby), you have some choices to make:

* Option 1: Do you want to play in this playground only when working on a certain project? If so, `cd` into the local repository where your project is located.  Then in your terminal enter `rbenv local 2.4.2`.  Just as bundler and git install files in your local repository to manage themselves, rbenv will install a local file called `.ruby-version` that will specify which version of Ruby to automatically use for this project.

* Option 2: Do you want to play in this playground only in your current terminal session? If so, type in your terminal `rbenv shell 2.4.2`. This overrides option 1.  But when you quit your current terminal session and open a new terminal session, this specification will no longer apply, and you’ll be back to using your operating system’s version of Ruby (or Option 1 if you’re in the applicable folder), unless you do the third option…

* Option 3: Do you want to play in this playground as a default setting every time you open up terminal session?  Then in your terminal, enter `rbenv global 2.4.2`.  Now, every time you open up a terminal session, you will be working in this version of Ruby.  That is, unless you’ve overridden it by taking advantage of Option 1 or 2.

I chose the third option.

See the readme.md [here](https://github.com/rbenv/rbenv) for more information.

**Step 7: Install some gems to make your life easier**

As noted above, when you are outside of a Ruby version manager like rbenv or RVM (so, in other words, when you are within the confines of your operating system’s Ruby), your operating system will tell you that you don’t have permission when you try to install a gem.  The way around this, without doing potentially dangerous stuff, is to engage a rbenv or RVM version of Ruby.  When you have a specific version of Ruby engaged through rbenv or RVM and you install a gem, that gem will be installed alongside rbenv or RVM, away from your operating system's Ruby.

There are cooler things you can do with rbenv/RVM and gems, but I think that covers the basic theory.  For what it’s worth, here are the gems that were installed on my operating-system-Ruby, after installing Xcode and before doing anything else (obtained from typing `gem list`)

```
> bigdecimal (1.2.0)
> CFPropertyList (2.2.8)
> io-console (0.4.2)
> json (1.7.7)
> libxml-ruby (2.6.0)
> minitest (4.3.2)
> nokogiri (1.5.6)
> psych (2.0.0)
> rake (0.9.6)
> rdoc (4.0.0)
> sqlite3 (1.3.7)
> test-unit (2.0.0.0)
```

***So, what other gems did I install in the course of all this, to make sure everything worked fine?***

* First, of course, prior to installing any gems, make sure you’ve got a rbenv or RVM version of Ruby active.

* Using `gem install <gem_name>` I started by installing gems that the curriculum had so far exposed us to: `bundler`, `rspec`, and `pry`.

* I also installed two other gems that Flatiron recommends ([here](http://help.learn.co/workflow-tips/local-environment/mac-osx-manual-environment-set-up-for-immersive-students)): `nokogiri` (which was already installed, perhaps by Xcode) and `phantomjs`.

* Flatiron also recommends installing the `learn-co` gem.  This allows you, from your terminal, to type familiar IDE commands like `learn test` and `learn submit`.  For more than a month I was stubborn and refused to install the gem, fearing it would basically make my terminal just like the IDE.  Then, a few weeks ago, I gave up and installed it.  Turns out, it does make your life easier to have `learn test` and `learn submit` at your fingertips.

* When I was using rbenv, it seemed to work hand-in-hand with a program called ruby-build, which in turn depends on the gems `openssl`, `libyaml`, and `libffi`.  I installed these via homebrew per the ruby-library [instructions](https://github.com/rbenv/ruby-build/wiki#suggested-build-environment).

* Lastly, I ran `gem update --system` to make sure everything was updated.

*****
<p />
Up next: [Part III - Making It All Come Together and Some Troubleshooting Tips](/coding/2017/12/07/local-developer-envionment-part-iii)
