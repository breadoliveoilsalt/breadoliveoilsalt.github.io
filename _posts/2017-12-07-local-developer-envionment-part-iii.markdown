---
layout: post
title:  "Lessons Learned Setting Up a Local Developer Environment, Part III: Making It All Come Together and Some Troubleshooting Tips"
date:   2017-12-07 12:00:00 -0400
categories: coding
---


**Part III -- Making It All Come Together and Some Troubleshooting Tips**

This post is the last companion of [Part I](/coding/2017/10/30/local-developer-environment-part-i) and [Part II](/coding/2017/11/29/local-developer-environment-part-ii).  If you've made it this far, thanks for going the distance.  I hope these have been of some help.  

*****
<p />
If you are a Flatiron Student, by this point, you should now have done enough to be able to work through lessons outside of the IDE.  As discussed below, you might still need it, but its role will be minimal.  Here is a general list of steps and pointers going forward.

* **On organizing:** It’s helpful to have a general local repository (aka, a folder) where you will work on your lessons.  This is where you can locally clone the lessons you fork. On my desktop, I simply have a `projects` folder, which in turn has sub-folders named after the various parts of the curriculum (`activerecord`, `sinatra`, etc.)

* **On forking a lab to work on it without the IDE:** When you start a new lesson, do not open up the IDE.  Instead, while on the webpage for a lesson, click the GitHub link to the right of the “Open IDE” button.  Then manually fork the lab on github.com and clone the lab to your local repository from the previous step.  See [here](http://help.learn.co/workflow-tips/github/how-to-manually-open-a-lab]) for how to manually fork a lab. Once I’ve forked a lesson and copied the relevant address to clone the forked lab, in my terminal I make sure to `cd` into my `projects` folder and then enter `git clone <pasted-address>`.  If you’re unsure what directory you are in, typing `pwd` will tell you (for “print working directory”).

* **On working in Atom:** After you’ve cloned the lesson to make a copy on your computer, use your terminal to `cd` into the root directory you just cloned.  (So, if you've just cloned `dynamic-orm-lab-v-000`, enter `cd dynamic-orm-lab-v-000`).  Feel free to create a branch with git and checkout that branch to work on it (`git branch yourbranch`, followed by `git checkout yourbranch`).  Then can enter `atom .` in your terminal  to have Atom open the folder (and current branch) in Atom.

* **But remember when typing `atom .` :** If you still have the IDE installed with Atom, a minor hiccup can occur here because the IDE is a version of Atom.  Entering `atom .` will simply open whichever version of "Atom" you had opened last -- Atom itself or the IDE.  So if you last had the IDE open, before typing `atom .`, find the Atom icon in your Applications folder and open Atom that way.  Then `atom .` should open Atom properly until the next time you open the actual IDE.  

* **Before running any tests, always run `bundle install`.** This will install the relevant gems you need to run `rspec`.  You must do this otherwise you will get all kinds of errors that seem to make no sense.  

* **If you are getting general errors that `rspec` or `shotgun` or any other commands won't run,** try prepending `bundle exec ` to your command -- for example, `bundle exec rspec`.

* **If, after running bundle install, you are getting errors about a particular gem (like `json` or `arel`),*** try running `bundle update <troublesome-gem's-name>`.  For example, I would get random errors about the `json` and `arel` gems that seemed to make no sense and have nothing to do with the code I was testing.  Running `bundle update json` or `bundle update arel` seemed to fix these problems.  See [here](https://github.com/flori/json/issues/253) for some reading on the issue.  BUT later I came to strongly suspect that the root of these problems was that I was working in Ruby v2.4.2, when I should have been using Ruby v2.3.1, which is what the IDE uses.  **So making sure you are in the same version of Ruby as the IDE should be your first line of attack** toward fixing such problems.  

* **If you have done some coding in your local environment but are getting wacky errors and want to jump to the IDE to see if that solves the problem:** It turns out that the IDE pulls its code from the master branch of your GitHub repository.  So, if you'd like the IDE to open up with your previous work already there:

* 1) If you're working in a branch, merge your branch into your master (`git checkout master`, followed by `git merge <branchname>`).

* 2) Push your local master branch to GitHub (`git push origin master`).

* 3) Make sure you have closed Atom.

* 4) Then, when you click on the blue "Open IDE" button in your lesson's webpage, the IDE will load up with your code in the text editor.

* 5) **BUT a warning about "fast forwarding":** Once you open your code in the IDE, if you make changes and enter `learn save` or `learn submit`, these commands will push your changes to your GitHub master branch.  If you then try to work on your local master branch prior to merging these changes into your local master branch, you will have divergent masters, which creates a "fast forwarding" problem.  I won't go into it much more here, but it's a real problem.  So if you leave your local terminal to try out the IDE, I'd recommend either staying with the IDE or never entering `learn save` or `learn submit` until you are ready to leave your local master branch behind.  Or better yet, spend the time to learn how to fix this using git commands.  You can read more about the "fast forwarding" conundrum [here](https://confluence.atlassian.com/bitbucket/git-fast-forwards-and-branch-management-329977726.html).

* **If you are stubborn like me and don't install the `learn-co` gem:**  To run your test and have them look like they do in the IDE when you enter `learn test`, instead enter in your terminal `rspec --format=documentation`.  However, to get a lesson's webpage  to recognize that you have passed all the tests, you need to open up your code in the IDE to run `learn test`.  Open up your local code automatically in the IDE using the steps immediately above, which push your code to GitHub prior to opening the IDE. To submit the lesson, you can type `learn submit` as usual, or make a manual pull request, described [here.](http://help.learn.co/workflow-tips/github/how-to-manually-submit-a-lab).  If, after typing `learn submit`, you get an error message that indicating that the learn submit is unsuccessful because it would publish a private email address, you will need to do a manual pull request.  It only takes an additional 30 seconds, so no big deal.  

*****
<p />
Thanks for stopping by.  If you're working on setting up a local development environment, chances are that the process will drive you a bit crazy at first.  Nevertheless, the process also gives you glimpses into what is going on under the hood.  These glimpses, in turn, become very helpful the further down the line you go.  The process is worth it.  Best of luck!  
