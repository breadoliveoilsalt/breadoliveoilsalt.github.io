---
layout: post
title:  "Lessons Learned Setting Up a Local Developer Environment, Part I: Background, Basic Installations, and Getting Git to Work"
date:   2017-10-30 12:00:00 -0400
categories: coding
---

**Part I -- Background, Basic Installations, and Getting Git to Work**

This post and the two that follow describe lessons I learned while setting up a local developer environment on my Mac. I'm currently enrolled as an online student at the Flatiron School (a coding bootcamp), so there are references here and there to programs and procedures specific to Flatiron students. Nevertheless, I suspect the steps described might be a good overview for anyone attempting to set up a local developer environment for the first time.

Full disclosure: there are three separate posts on this topic in large part because I encountered so many snafus to troubleshoot along the way.  I hoped that by chronicling the details of what I did and the problems I encountered, these posts might help someone in a similar situation avoid such problems.  Unfortunately, the nature of the beast means you'll probably run into snafus too, but for what it's worth, none of mine did any permanent damage, and all of them were solvable with a bit of internet research.  At the end of the day, wrestling with them has been quite a learning experience, and I’m glad I started tinkering.

Part I of this blog post deals describes some background for those who aren't familiar with the Flatiron School's "Learn IDE."  Then Part I goes into some of the first steps toward getting everything to work, including making sure Git is synced up with GitHub.


**Background**


Those of us doing Flatiron’s web developer program online work through the curriculum using something called the Learn IDE 3 (aka, the “IDE”).  It’s a beefed up version of Atom’s text editor.  When you open up the IDE, the screen is split between a text editor part and a terminal for entering commands.  To open the IDE and use it to solve a coding lab, we head to the webpage on [learn.co](http://www.learn.co) describing the lab, click a blue button on the page, and the IDE loads up, proving us with the files we need for the lab, some tests, maybe even a little starter code.  It’s a very controlled environment.  Any necessary gems are pre-loaded.  There’s no need to use git-based commands to load or push anything to git, and no need to worry about random dependencies you may not be aware of.  It’s all right there.  As far as coding playpens go, it’s quite nifty and an impressive feat by the people who put it together.  

When I started writing this, I was somewhere the curriculum toward the end of the Procedural Ruby section.  While working up to that point, I had been feeling greedy.  I wanted a bit more than the IDE could offer.  For example, the IDE doesn’t let you “save all” when you have a bunch of editor tabs open.  I missed being able to use git branches, and I felt the bit of git commands I knew growing rusty without having to use them.  And perhaps most of all, I actually wanted to use the terminal as a stand-alone program -- in part so I would get used to it, and in part so my terminal window could stretch out without having to share space with a text editor window.

For its in-person students, Flatiron’s internal website has a useful help page on [the basics of setting a working environment](http://http://help.learn.co/workflow-tips/local-environment/mac-osx-manual-environment-set-up-for-immersive-students).  There’s a big warning on this page, though: “Note for online students: We do not provide support for environment issues if you set up a local environment using a Mac.”  Also, the first step on the help page is to fully uninstall the Learn IDE.  So, needless to say, this page isn’t quite for people in Flatiron’s online developer program.

My goal ultimately became to see if I could have the best of both worlds: was there a way I could set up an independent local developer environment, but also keep access to the Learn IDE if I needed it? This blog post and the two that follow basically chronicle how I tinkered with things to get to that goal. By the time I finished putting the bulk of these posts into narrative form, I was somewhere in the Rack portion of the curriculum, still with the IDE on my computer but using my local developer environment, and for the most part, this experiment has gone fine.  

The steps below are in the imperative form -- that is, “do this/do that” -- but that’s merely for ease of readability. Please don't think of these steps as gospel-truth; unfortunately, snafus will surely arise.  But, for what it's worth, dealing with the snafus have been a heck of a learning experience for me, and grappling with them has helped me to understand a little more about what is going on under the hood.  

Also, here are some defined terms that appear throughout the posts and their meanings:

* “**IDE**” refers to the Learn IDE 3.

* “**Atom**” refers to actual Atom text editor, not the IDE.

* “**The terminal**” refers to the Terminal application on your Mac, not the terminal embedded in the IDE.  

Here we go...


**Step 1: Plan a strategic retreat**


Back up your Mac in case things go south and you need to undo everything completely.  Just sayin’.


**Step 2: Get a text editor.  If it's Atom and you still have the IDE installed, prepare for Atom's identity crisis**


If you're going to move away from the IDE, you'll need to download another text editor, like Atom or Sublime Text.

Prior to downloading the IDE, I had plain old Atom on my Mac.  After downloading the IDE, however, Atom seemed to have an identity crisis.  Sometimes when I would type `atom .` in the terminal to open Atom and load it up with my current directory, the IDE would pop up instead.  Not wanting to mess around with Atom itself or uninstall anything yet, I figured why not give Sublime Text a shot (this is the text editor that the instructors use in some of the older videos available through the curriculum).  [Here](http://https://www.sublimetext.com/3) is where you can download the latest version.

***But a few lessons learned along the way:***

* I figured out the problem behind Atom's identity crisis:  when you type `atom .` in the terminal, the terminal opens up whichever program you had open last -- Atom itself, or the IDE.  So if the IDE is loading when you want Atom, make sure the IDE is completely closed out, open Atom via its icon (likely in your Applications folder), and then type `atom .` in your terminal.  All should be well.  

* It turns out that the latest edition of Sublime Text is no long free.  After a few uses, it reminds you of this fact.  

* You need to do some playing around to get Sublime Text to open up with a terminal command, `subl .` (the equivalent of `atom .`) .  See [here](http://https://stackoverflow.com/questions/16199581/open-sublime-text-from-terminal-in-macos) and [here](http://http://olivierlacan.com/posts/launch-sublime-text-3-from-the-command-line/).

***So*** at the end of the day, despite the hiccup with Atom's identity crisis, I’d recommend downloading the latest edition of plain-old Atom, and using that as your go-to, non-IDE text editor.  


**Step 3: Install Xcode on your Mac**


Trying to set up a local developer environment was my first introduction to Xcode.  It’s a developer tool from Apple for making things run on Mac OS or iOS.  I haven’t played around with Xcode much yet, but downloading it installs things you’ll need to use on your terminal, like git.  You can download Xcode from the App Store.  The application is huge, takes a lot of time to download and install, and takes up a lot of hard disk space.

***But a lesson learned along the way:***

* You have some options when it comes to installing Xcode.   Some on the internet say you don't have to download the storage-hungry app from Apple's App Store.  Instead, there is a more storage-friendly alternative, which is to download the basics Xcode tools via a terminal command line.  See [here](http://https://www.moncefbelyamani.com/how-to-install-xcode-homebrew-git-rvm-ruby-on-mac/), for example.

At some point in dealing with snafus, I did both, installing the Xcode app from the App Store first.  I’d recommend downloading the basic XCode tools via the command line (as described in the link immediately above) and seeing if that works for you.  If you run into problems while in the middle of this journey, then perhaps try downloading the whole Xcode app.  


**Step 4: Get git and GitHub synced up on your terminal**


Prior to installing Xcode, git was not available on my terminal.  Typing any git command (like `git status`) got me an error saying git was not there.  After installing Xcode, that was no longer the case; git was there and available. The problem was, having worked through a good portion of the Flatiron curriculum with the IDE already, I had all these GitHub repositories online, associated with a previously-established GitHub account.  I wanted my terminal to be associated with the same GitHub profile that the IDE spoke to.  In other words, I needed a way of telling my GitHub account that *me* was actually *me* when I tried to push a repository to GitHub from the terminal.

***A few lessons learned along the way:***

* The key to letting GitHub know who you are from your terminal is setting up an SSL key (no pun intended).  My limited understanding is that an SSL key is an encrypted, one-factor log-on authentication.  Apparently, much of the internet's functionality (especially networks and cloud storage) relies on such things.  It's like having your computer carve a physical, complicated key.  You have tell GitHub that when your terminal knocks on GitHub’s door, your computer’s key works to open the door to your GitHub profile.  Once you’ve got the door open, you can push things to your GitHub remote repository just like the IDE but from your own terminal.

* **To set up your SSL key:**

  * 1) Start [here](http://https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/), as an overview.

  * 2) Then, as that page suggests, go through the following steps.  You should make sure you have a general sense of what is going on, but at the end of the day, entering the commands at your terminal as these pages recommend will get you where you need to go.  


  * 2a) Check for an existing SSL key. See [here](http://https://help.github.com/articles/checking-for-existing-ssh-keys/).

  * 2b) Generate a new SSL key.  See [here](http://https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/). **A note on setting up a password:** Some sources I read were ambivalent about setting up a password here.  I did.  But then every time I wanted to push or pull from git, I had to enter my password, which got old fast.  The solution, so you don’t have to enter your password all the time, is to add your SSH key to the “SSH-agent,” which is on the bottom half of the page in the link above.  

  * 2c) Add the new SSH key to your GitHub settings. See [here](http://https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/).

  * 3) Once you are done, test if everything is working:

  * 3a) Create a local repository on your computer.  For example, in your terminal, navigate to your desktop with `cd ~/desktop` and create a "testdirectory" by entering `mkdir testdirectory`.  Enter `cd testdirectory` to navigate to your new local repository and type `git init` to let git know locally that it should keep track of your files within testdirectory.  

  * 3b) Then create a test file.  For example, in your terminal (still within testdirectory), type `touch testfile.md`.  Begin a git commit by entering `add .`  and then complete the commit with the message of your choice, something like: `git commit -m “setting up testfile”`

  * 3c) Create a repository on GitHub to correspond with your local repository, "testdirectory."  That is, once you are on a GitHub webpage and logged in, in the upper right corner, click on the + and create a new repository called testdirectory.  After clicking “Create repository,” take a look at the directions there.  You’ll want to do what it says under the second option: “…or push an existing repository from the command line.”  So back on your terminal, enter something like `git remote add origin git@github.com:<your profile>/testdirectory.git` . This will link your local repository with your online one.

  * 3d) In your terminal enter: `git push -u origin master` to push your local repository's changes (that is, the creation of testfile.md) to your remote repository on GitHub.  Then check your testdirectory on GitHub.  If “testfile.md” is there, then you’ve successfully linked your online GitHub profile with your ability to work on your own terminal, without the IDE.  

**An important note on publicizing email addresses:**

* If git prevents you from pushing or pulling and is throwing error messages that your action would cause a private email to be published in violation of your GitHub settings, then you need to change the email address associated with git at your terminal (remember that git is separate from GitHub).  Follow the instructions [here](http://https://help.github.com/articles/about-commit-email-addresses/) and [here](http://https://help.github.com/articles/setting-your-commit-email-address-in-git/).  These instructions tell you how to change your git email at your terminal to a long, do-not-reply email address that GitHub provides.  Then all should be fine.  The do-not-reply email address can be found when you are logged into GitHub under “Settings” → “Email”.     

* Setting your email to the do-not-reply email address seems to be good practice in general; otherwise, it seems you [risk exposing your email address](https://www.sourcecon.com/how-to-find-almost-any-github-users-email-address/).  To see if your email address is being exposed when you commit, enter git log in the terminal and checkout what it says for the email address there.  If you see your do-not-reply email address from GitHub, then you should be good to go.   

*****
<p />
That's it for now.  Up next: [Part II - Start Managing What You Install Via the Terminal: Homebrew, Ruby Version Controlling, and Gems](/coding/2017/11/29/local-developer-environment-part-ii).
