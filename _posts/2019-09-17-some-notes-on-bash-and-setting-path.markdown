---
layout: post
title: "Essential & Relevant: A Unit Test Balancing Act"
authors: ["tony-distinti"]
tags: ["Apprenticeship", "Coding", "macOS"]
social-img:

---

It's funny how going back to basics can make your realize what you've been taking for granted and don't understand.  

On the first day of my apprenticeship at 8th Light, I was presented with a wiped MacBook Pro, ripe for configuration and the installation of various libraries, Ruby gems, etc.  In the midst of this set up, I got the urge to play around with splitting Terminal windows.  I went hunting for the latest version of [GNU Screen](https://www.gnu.org/software/screen/), which would allow me to split a single Terminal window, and installed it via [Homebrew](https://brew.sh/).  I quickly hit a wall, however, finding that couldn't get this new version of GNU Screen to run when I typed the launch command `screen`.  Instead, an older version of GNU Screen would run, one that came pre-installed with the Mac OS and didn't have the functionality I wanted.  

I suspected that this was happening because the Terminal didn't have access to the command to launch the latest version of GNU Screen, and something in the back of my brain told me I had to then play with the `PATH` associated with the Terminal session.  I had done this in the past and had a basic understanding of the process, but when going down proverbial internet rabbit hole to refresh my memory, I quickly realized how much I had forgotten.  More poignantly, when tempted to get quick fixes by copying and pasting code snippets from [Stack Overflow](https://stackoverflow.com/), I realized *how much I didn't know* and *how much I couldn't articulate* about what was going on with the Terminal, setting the `PATH`, etc.  

A few days later, when thinking about what to write for this blog post, I thought it might be a great opportunity to try to begin filling those gaps and to share what I came across in case others have similar questions.  The discussion below starts off with basic questions about the Terminal and Bash.  Then it moves on how the Terminal window's `PATH` variable is set and how its particular formatting works.  The answers are overly simplistic, but hopefully helpful to anyone who might be reading.  The sources I used are in the links.  

By the way, the problem I suspected with GNU Screen was completely wrong.  The solution to the actual problem is described [at the bottom](). LINK AND DESCRIPTION TO COME.

**So, what is the Terminal on a Mac?**

The [Terminal](https://support.apple.com/guide/terminal/open-or-quit-terminal-apd5265185d-f365-44cb-8b09-71a064a42125/mac) is a programg for opening up a *shell* on your computer.  

**Ok, so what's a shell?**

A [shell](https://www.tldp.org/LDP/Bash-Beginners-Guide/html/Bash-Beginners-Guide.html#sect_01_01) is a program that gets a computer to do something based on commands that a user types.  The shell *interprets* the user’s commands.  It is, fundamentally, a “command-line interface” for interacting with a computer.  There are different types of shells, each of which accepts different commands from the user.  

**Does this mean that when I open up Terminal, it opens up a particular type of shell?**

You got it.  At the time of writing this, Terminal by default opens up a particular shell called [Bash](https://www.gnu.org/software/bash/manual/html_node/What-is-Bash_003f.html).  The name "Bash" is an [acronym for Bourne Again Shell](https://www.gnu.org/software/bash/manual/html_node/What-is-Bash_003f.html).  Some of Bash's commands and features [are different from those found in other shells](https://www.tldp.org/LDP/Bash-Beginners-Guide/html/Bash-Beginners-Guide.html#sect_01_02).  

**Can I set Terminal to open up another type of shell, if I'd prefer to use that other shell?**

Yup.  See [here](http://osxdaily.com/2012/03/21/change-shell-mac-os-x/), for example.

**So assuming I use Bash, what's up with `PATH`?  Why is it important?  Why are there Stack Overflow questions about changing it?**

Before we get there, let's take a step back and ask some other questions.  

**What happens when Bash opens?**

The way Bash appears and its functionality depend on certain *[environmental variables](https://linuxhint.com/bash-environment-variables/)* being set as it opens.  I think the simple way to think about the environmental variables is that they are settings that can affect Bash, and perhaps applications you open and use through Bash.

There are environmental variables for many differing things, including the prompt you see next to your cursor at the terminal.  On a Mac, every time a new Terminal window is opened with Bash, many of these environmental variables are set by default.  You can see a list of these variables and their default if you type `man bash` (for Bash Manual) at the Bash prompt.  

**What does this have to do with `PATH`?**

If you've ever used the `cd` command in Bash, you know that you can "enter into" different directories (aka, folders or repositories) with Bash and enter commands that affect this particular directory.  For example, if you wanted to use the Vim text editor to create and edit a file called  `README.md` in your current directory, you could type `vim README.MD`, and Bash would know to open up Vim, and once the file was saved with Vim, `README.md` would be in saved in your current directory.  

You could take this same action from any directory you `cd`'d into with Bash.  No matter which directory your Bash prompt was in, Bash would know that when you typed `vim README.md` you wanted to open up an application called Vim, even though Vim is not located in your current directory.  How does Bash know to do this?

This is where `PATH` comes in.  `PATH` is, fundamentally an [environmental variable](https://scriptingosx.com/2017/04/on-bash-environment-variables/) that lists a set of directories for Bash to check when a user enters a command that is not located in the user's current directory.  So when a user begins a command with `vim`, but Vim is not in the current directory, Bash says to itself, "Let me see if I can find Vim in the directories listed in PATH before I come back to the user with an error because I don't know what to do."

Want to see which directories are in your current path?  At the Bash prompt, type `$PATH`.  You'll see a response from Bash that might look something like this:

`-bash: /usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin`

**What does all that mean??**

The stuff after `-bash: ` is a list of the various directories in the `PATH` that Bash might check when looking to interpret a command.  Bash's output above is hard to read at first because the different directories are separated by colons.  So in other words, the line above is telling you that the following directories are in Bash's `PATH`:

* `[user hard drive]/usr/bin`
* `[user hard drive]/bin`
* `[user hard drive]/usr/sbin`
* `[user hard drive]/sbin`
* `[user hard drive]/usr/local/bin`

The order of the list is important.  Bash will check the directories [in the order in which they are listed](https://unix.stackexchange.com/questions/26047/how-to-correctly-add-a-path-to-path).  

Please note: the directories above are generally "hidden" on a Mac.  So to see them from Bash, you'll have to be in your top most, "hard drive" directory and use the command `ls -lah`. If you are looking at a Finder window to see a visual representation of folders, make sure you are in a window showing the contents of your top most directory and then hit `command` + `shift` + `.` to see the hidden directories.    

Once you're able to see these "hidden" directories, go ahead and poke look around!  You'll see that Bash depends on its PATH to interpret even fundamental commands like `ls` (in `[user hard drive]/bin`) and `cd` (in `[user hard drive]/usr/bin).

**Ok.  But now let's say I've installed a command line tool via Bash and the command to launch the program is not working.  Stack Overflow is telling me to modify my `PATH` by pasting some text in a file called `.bash_profile` or `.bashrc`.  What's going on here and what does the inserted text mean?**

Let's use a real world example.  [Here](https://stackoverflow.com/questions/10343834/how-to-modify-path-for-homebrew) are some Stack Overflow responses dealing with modifying the `PATH` after installing a program via Homebrew.  One of the answers suggests adding the following line to the user's `.bash_profile`:

`export PATH="/usr/local/bin:$PATH"`

To understand this suggestion, the first thing I'd explore is...

**What is `.bash_profile`?**

When Bash loads, it can look to several different sources for instructions and the values of particular environmental variables.  These sources include files such as `.bash_profile` and `.bashrc`, which are located in the directory `[user hard drive]/Users/[your user name]`.  Which one should you use?  There are numerous discussions out on the internet on this topic, such as [here](https://medium.com/@kingnand.90/what-is-the-difference-between-bash-profile-and-bashrc-d4c902ac7308), [here](https://apple.stackexchange.com/questions/51036/what-is-the-difference-between-bash-profile-and-bashrc), and [here](https://unix.stackexchange.com/questions/26047/how-to-correctly-add-a-path-to-path).

At the current moment, as my rudimentary knowledge on this topic is developing, I think the safest and easiest bet is to add the text to `.bash_profile` if you're working on a Mac, since Terminal's default is to look to `.bash_profile` every time a new Terminal window with Bash is opened.

**Ok, so it looks like `.bash_profile` is now setting the `PATH` when my Bash session opens.  But what does this like of code really mean? For example, what is `export`?

`export` is a command that says: what ever follows on this line of code [should apply to all of Bash's child processes](https://superuser.com/questions/153371/what-does-export-do-in-bash) from that point on.  In overly-simplistic plain language, I think this means: for commands executed with Bash from now on, those commands will look to and depend on whatever is `export`ed.  

**What happens next?**

`PATH` is a Bash environment variable and discussed above, and the `=` operator sets what the `PATH` should be from now on.

**Ok, so I get that colons separate distinct directories when the `PATH` variable is being set, and `.bash_profile` is now defining the `PATH` as the BASH session loads.  But what's up with `$PATH` at the end of the line? In particular, why is there a dollar sign?**

Let's take a look again at the line of code suggested by the Stack Overflow answer, after the `export` command:

`PATH="/usr/local/bin:$PATH"`

As it turns out, with the [Bash language](https://stackoverflow.com/questions/28693737/is-bash-a-programming-language), a variable is assigned a value by simply writing the variable name, followed by a `=` operator, and then the value .  But to *access* the value assigned (this access is generally called "[parameter expansion](https://opensource.com/article/17/6/bash-parameter-expansion"), the variable needs to be preceded by `$`.  See, for example, the discussions [here](https://stackoverflow.com/questions/18658190/bash-when-to-use-in-front-of-variables) and [here](https://opensource.com/article/17/6/bash-parameter-expansion).  In other words, the `$` is analogous to a getter method for the variable that follows or an operator for string concatenation.  

So this...

`PATH="/usr/local/bin:$PATH"`

...is essentially saying:

1. We're modifying the `PATH` environmental variable.
2. From now on, the first directory that Bash should look to when executing a command is `[user hard drive]/usr/local/bin`.  This is where Homebrew installs applications, and Bash knows to look there first because it is now the first directory listed in the `PATH`.
3. If Bash hasn't found a command in the Homebrew installation directory, then Bash should look the default directories that have been assigned to the `PATH` variable prior to this new assignment.  By using a dollar sign (`$PATH`), the code says to add the value of the `PATH` variable (the list those default directories) here.  These default directories likely include:

* `[user hard drive]/usr/bin`
* `[user hard drive]/bin`
* `[user hard drive]/usr/sbin`
* `[user hard drive]/sbin`
* `[user hard drive]/usr/local/bin`

So, in sum, once `.bash_profile` modifies the `PATH`, the directories that make up the `PATH` and that Bash will look to when executing commands are, in order:

* `[user hard drive]/usr/local/bin`
* `[user hard drive]/usr/bin`
* `[user hard drive]/bin`
* `[user hard drive]/usr/sbin`
* `[user hard drive]/sbin`
* `[user hard drive]/usr/local/bin`

I hope all this helps make clearer what is going on under the hood the next time you might have to modify your Bash `PATH`, or perhaps even write a [Bash script](https://linuxconfig.org/bash-scripting-tutorial-for-beginners).  Thanks for reading!  

-----------

**__Solution to Launching the Latest Version of GNU Screen__**

After installing the latest version of [GNU Screen](https://www.gnu.org/software/screen/) via [Homebrew](https://brew.sh/) and restarting my Terminal session, typing in `screen` would launch the older, pre-installed version of GNU Screen.  The problem was simply that the command to launch the new version of GNU Screen required the version number.  That is, I had to type `screen-4.6.2`.  I only discovered this by chance after doing a dive into where Homebrew tends to install files: the hidden folders under `[user hard drive]/usr/local/Cellar` and `[user hard drive]/usr/local/bin`.