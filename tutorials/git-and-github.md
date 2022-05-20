---
layout: post
title: "Intro to Version Control with Git"
permalink: /resources/tutorials/git
---

## git?
Git is a version control system which lets you publish your work online and save
versions of your code as you progress. Saving (or 'committing') work as you go
means that you never really lose earlier versions: you can return to a previous
state at any moment.
Github repos are also a great place to store the tutorials you complete as you learn.


(If you are not comfortable with the basics of your terminal, I suggest you complete [this exercise](https://learnpythonthehardway.org/python3/appendixa.html) first.)

This tutorial will cover the steps you should take every time you start a new
project, and then how to manage your project effectively.

## quick jump
<ol start="0">
<li><a href="#tldr-summary-of-commands">Summary of commands</a></li>
<li><a href="#part-1-setup">Setup git</a></li>
<li><a href="#part-2-create-your-project-locally">Create local project</a></li>
<li><a href="#part-3-create-the-remote-version">Create remote version</a></li>
<li><a href="#part-4-connecting-the-two">Connect local to remote</a></li>
<li><a href="#part-5-first-commit">First commit</a></li>
<li><a href="#part-6-pushing-to-the-cloud">Push to the cloud</a></li>
<li><a href="#part-7-writing-and-publishing-our-first-program">Subsequent commits</a></li>
<li><a href="#part-8-taking-notes">Note-taking in Markdown</a></li>
<li><a href="#part-9-getting-a-copy-of-your-work-from-the-cloud">Cloning</a></li>
<li><a href="#part-10-pulling-updates-from-the-cloud">Pulling</a></li>
<li><a href="#part-11-creating-a-work-in-progress-branch">Branching</a></li>
<li><a href="#part-12-merging-branches">Merging</a></li>
<li><a href="#part-13-dealing-with-merge-conflicts">Merge conflicts</a></li>
</ol>

## tldr summary of commands
For if you have already read the whole way through this tutorial and are just
back here to remind yourself of the commands.
(If this is your first time here, please jump to [Part 1](#part-1-setup).)

- do not type the bits after `#`; they are just notes
- words inside `<>` are placeholders, replace them with your details

```sh
cd ~/code && mkdir <new-project-name> && cd <new-project-name> # create project directory
git init # initialise git tracking for your local version

# log into your github account in a brower and create a new repository by clicking the plus at the top right

git remote add origin <url of your repo online>

echo "<short project description>" >> README.md # optional, you can also do this later or write a longer one in a text editor

git status # see that there is a new untracked file
git add README.md # track that file
git status # see that the file is now tracked
git commit -m "first commit - adds readme" # commit to the state of your project
git log # see the (short) commit history of your project
git push -u origin main # push your local version to the cloud

# refresh your browser and see your online remote version has been updated

# write some code

git status # see the new code you have written is untracked
git add <file(s) you want to track>
git status # see the new file(s) are now tracked
git commit -m "<brief message about what has changed in/been added to your project>" # commit to the new state/version
git log # see your growing history
git push # push your new stuff to the cloud

# switch to a new computer / accidentally delete your local copy
cd ~/code
git clone <url of your repo online>
cd <project-name>

# commit some things in another version of the project, or using the online Github editor
git pull # get those changes in your local version

# can't pull because of local changes?
git stash
# or if those changes are not important
git checkout <file path> # WARNING changes will be lost forever, be certain!
git stash pop # recover stashed changes

# changed your mind about a commit?
git reset head^ # if the commit has not been pushed yet
git revert <commit sha> # if the commit has been already pushed. use 'git log' to get the SHA (random string of characters which identifies the commit)

# work from a new branch
git checkout -b <new-branch-name>
git push origin <new-branch-name>
git checkout main
git merge <new-branch-name>
git branch -d <new-branch-name> # delete local
git push origin --delete <new-ranch-name> # delete remote
```

## Part 1: Setup

**Setup Steps:**
1. Type the following into your terminal and press Enter:

	```sh
	git --version
	```
	If that returned something like `command not found`, you can download and install
	git [here](https://git-scm.com/download/mac).
  
1. Sign up for [Github](https://github.com/) if you have not already done so, and complete their beginner's guide.

1. Run the following two commands to save your login information to your computer. Every time you "push" (save) code to git, these credentials will be used to identify you.
	(_The things in caps are placeholders, **please use your actual information**._)

	```sh
	git config --global user.email "YOUR EMAIL ADDRESS"
	git config --global user.name "YOUR GITHUB USERNAME or ACTUAL NAME"
	```

1. Run `ssh-keygen -t rsa -b 4096 -C "YOUR EMAIL ADDRESS"`. (Hit return to any prompts until complete.)
	This generates two files: `id_rsa.pub` and `id_rsa`. These essentially act as a lock and key which can
	be used to ensure that only you can save your work in your user area on Github.
1. Start the SSH agent with `eval "$(ssh-agent -s)"`. ([SSH](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys) is a form of security)
	(This turns on the program which is going to manage your 'key'.)
1. Give your 'key' to the `agent` mentioned above: `ssh-add ~/.ssh/id_rsa`.
1. Copy your public key to the clipboard:

	```sh
	# either
	pbcopy < ~/.ssh/id_rsa.pub
	# or
	cat ~/.ssh/id_rsa.pub | pbcopy
	# these commands are the equivalent of looking at the file and pressing <Command-C>
	```

1. Login to Github in a browser. Go to `Settings > SSH and GPG keys > New SSH key / Add SSH key`.
	(This is setting up the 'lock' in Github.)
1. Fill in the form fields with a name and your copied public key and click save.

## Part 2: Create your project locally
Your project will exist in two locations: the `local` version and the `remote`
version. The local version is the one which exists on your computer. The remote
is the version stored in the cloud on Github.

So let's begin by opening `iTerm` or `terminal`, and creating the local version.

**Steps**:

1. Chose a place on your computer to create your project. Most coders tend to
	keep all their projects inside a directory called either `code` or
	`workspace`. This directory is usually in their user `home` directory.
	If you already have a directory to store all your projects you can skip ahead
	to step 2.

	You can get to your `home` directory by running `cd ~` or simply `cd`. When
	you are there you can run `ls` and you should be able to see the folders
	you are used to seeing in `Finder` (`Documents`, `Downloads`, `Applications`,
	etc.).

	When you have decided on the name to call you main coding directory, create it
	and change into it.

	```sh
	mkdir code
	cd code
	```
1. Now we have a dedicated folder to store all our projects in, we can go ahead
	and create a directory for our first project. We are going to be creating a
	simple `hello-world` terminal program, so let's call our project that:

	```sh
	mkdir hello-world
	cd hello-world
	```

	Run `pwd` to check that you are in the right place. You should see `/Users/<your-name>/code/hello-world`.

1. Next we want to use `git` to track this project as we build it. It's best to
	do this from the start. Run `git init` and you should see `Initialized empty
	Git repository in /Users/<your-name>/code/hello-world/.git/`.

	_**note**: It is important that you always check that you are in the right_
	_place before you start working with git. Use `pwd` to see you are in the_
	_correct project directory, and then `ls` to ensure that only the files that_
	_are related to that project are visible. (There should be none if you have_
	_only just created that directory.) Any files in that directory could_
	_end up being stored publicly in the cloud, so take care not to include_
	_anything which is sensitive or private._

## Part 3: Create the remote version
Now you need to setup the corresponding project in the cloud.

**Steps**:

1. Open your browser and go to [Github](https://github.com).
1. Once you have logged in, click the plus sign at the top right of the page,
	and select `New repository` at the top of the drop down.
1. In the `Repository name` section write `hello-world`.
1. Click `Create repository`.

And that's it. Now we need to link the local and the remote.

## Part 4: Connecting the two

After we clicked `Create repository`, Github gave us a helpful list of commands
to run, so that is what we are going to do. As this is not an existing repo,
but a brand new one, we are going to use the commands suggested in the first
section. **Note**: We are going to be running these commands in a slightly
different order, so don't go copying and pasting the whole lot.

**Steps**:

1. Creating the link between the local and the remote is very easy: you simply
	need to give the local version the URL of the remote version.
	(You can copy and paste the URL from the 5th line of the commands which Github
	gives you.)

	```sh
	git remote add origin https://github.com/YOUR_USERNAME/hello-world.git
	```

	What is this command doing? The `remote` and `add` are self-evident, given we
	are trying to add a link to a remote repository. `origin` here is the shorter
	name that your computer will use to identify that the remote repository the URL
	points to is the main (original) version of this project.

	If you run `git remote -v`, you'll see that your project URL is saved with the
	`origin` tag.

## Part 5: First commit
**Steps**:

1. The first thing we are going to do is create a README.md. READMEs are a very
	old convention in computing and the clue to what they do is in their name.
	Every project comes with a file which in big, eye-catching capitals, invites
	users of the program to read the file first, before trying to use it.
	This was very important back in the 60s and 70s when computers were instructed
	by punch cards and the whole process of getting a computer and program to run
	was very involved and laborious: you needed a good set of instructions.
	We still use READMEs for the same thing today. They serve as descriptors and
	introductions for projects and what they do, and offer instructions for how
	to run and use the program. On Github, the README.md always serves as the
	landing page in a repository.

	As our project is not really going to be doing much, we are just going to create
	a short README explaining what it does. We can create a new file and write to
	it with just one command in the terminal. Github is telling us how to do that
	with `echo` (`echo "# hello-world" >> README.md`). We saw the `echo` command
	when learning about the command line: it is a command which can print words
	in the terminal. In this command, however, we are telling it that instead of
	just repeating the word back to us in the terminal, we would like to write that
	line into a file. If the file does not exist it will create it, and if it does
	exist, it will append the line to the end of the file. (to overwite an existing
	file completely we would switch `>>` for `>`.)

	So we are going to use `echo` to create a file with just one line in it:

	```sh
	echo "# learning about git" >> README.md
	```

1. Now that we have created the first file in our project, we want git to track
	it.

	If you run `git status`, you should see that git is aware of a new file in the
	directory, but that it is still `untracked`. Git has given us a hint as to how
	we can start tracking files: `use "git add" to track`.

	So let's add our README: `git add README.md`

	Now when we run `git status` again, we can see that (this time in green), git
	knows that we want it to track a new file.

1. But we are not done yet. Git knows that we want to track this file, but we haven't
	yet confirmed that we want to `commit` to this version and save the current state. 
	Are we happy that our project is in a good state that we want to share with
	the world?

	Right now we just have a README so we don't have much to consider, although if
	you want to change your README, you can open it up, write more, and run `git add`
	again. (You have to run `git add` whenever you change a file or git won't save
	those alterations for you.)

	Our README is in a good enough state, so now we are going to commit to that.
	When we make a `commit` we have to add a clear, concise message about what
	is new about this version or state. This is very useful when going back over the
	history of a project. Coders have to read old commit messages every day to quickly
	figure out what a section of code does and why it was written.

	To commit your README run:
	```sh
	git commit -m "first commit, adds readme"
	```

	After that run `git log`. You should see that today, you made your first commit
	in which you added a README.

## Part 6: Pushing to the cloud
**Steps**:

1. Now we need to make the remote version match the local version.
	To do that, we have to `push` our commits to our remote:

	```sh
	git push -u origin main
	```

	We know what `push` and `origin` mean, but what are `-u` and `main`?
	Let's start with `main`. If you run `git branch`, you should see `* main`.
	The git workflow is based entirely around branches. Every project has one main
	branch, `main`: this is the version of your project which must always be kept
	in a fully working state. You should never commit broken or in-progress code
	to `main`.

	If you have any new ideas, or experiments you want to test out, they should be
	built on a parallel branch. This branch is based upon `main` but will not
	affect it, meaning that people who come to your project will have access to a
	working version you can be proud of. Branches are also useful for when you don't
	quite have time to finish something that day, but you still want to save your
	work for later.

	When your experiments work out, or you finish that new idea, you can pull those
	commits onto `main`, and carry on normally.

	We will go over how to work with more branches later; for now let's just stick
	with `main`.

	`-u` stands for `upstream`. The `upstream` version is the main source of your
	project. You may think the main source is your computer but no. As your project
	is publicly viewable, and as you may spill coffee on your computer at any moment,
	the main remote repo is the `upstream`. Your computer holds a `downstream` version.
	The work you do is done `downstream` and is then pushed to the official `upstream`.
	A project can have many `downstream`s, and this is how many people can collaborate
	on the same project: by regarding the `upstream` as the source.

1. Once you have pushed, you can go back to Github and refresh that new project page.
	You should see your README instead of the git commands.

	Congratulations! You have set up your project and are now ready to get to work.

## Part 7: Writing and publishing our first program
We are going to be writing some actual code now, and learning some more about the
command line and how your computer runs programs.

**Steps**:

1. Complete the first [Hello World](https://github.com/fouralarmfire/square-one/blob/master/tutorials/hello-world-1.md#hello-world-1) exercise.
1. When your `hello` program is working (i.e. you can run `./hello` in the terminal
	and see `Hello World!` printed back to you), run `git status`. You should see
	that there is a new untracked file in your project.
1. Run `git add hello` to track your program.
1. Run `git status` again to verify that it is being tracked.
1. Run `git commit -m "new 'hello' program - says Hello World"` to commit your new
	project state.
1. Run `git log` to see your growing project history.
1. Finally we can run `git push` to send our code to the remote version.

	_**Note**: this time we did not need to run the push command with `-u origin main`._
	_That is because git stored the information about which upstream branch this local one is mapped to._
	_From now on, for this branch, all we need to say is `git push`._

1. You can now navigate to the online repo in your browser, and see your `hello` program.


## Part 8: Taking notes

Since we are learning new things, it would be handy to use the README of our repo
to document what we have learned after each section. The `.md` extension stands
for markdown and you can learn how to render documentation using markdown [here](https://guides.github.com/features/mastering-markdown/).

**Steps**:

1. Open your README.md in a text editor. 
1. Summarise what you have learned so far.
1. Commit and push your changes.

	```sh
	git add README.md
	git commit -m "update readme: parts 1 to 8 notes"
	git push
	```

From now on, for every project or tutorial you do, take notes on what you learn
like this so that you keep them together with the code you write.

## Part 9: Getting a copy of your work from the cloud

So, your shiny new program is now safely stored in the cloud. But what if you want to carry on working
from a different computer? What if you want to work on it with a friend and they need a copy? What if
you accidentally erased your entire `~/code` directory and need to get all the contents back again?!
(Full disclosure: I do that last one a lot!)

For this we use `git clone` which does pretty much what you would expect: it creates a clone of project from the cloud
at the point you tell it to. If you don't have another computer to test this on don't worry, you can have as many copies
of the same project as you like, so we will just clone to your current computer.

**Steps**:

If you feel confident, and have committed and pushed everything, you can follow the steps in section **A**, if not
please go to section **B** just below.

**A**) 

1. Move out of your project directory.* If you run `pwd` you should see `/Users/<your name>/code/hello-world`, so to move out type `cd ..`.
		If you run `pwd` now you should see that you are in `/Users/<your name>/code`. If you get lost, you can always find your way back by
		using `cd <path>` and `pwd`.
1. Delete your `hello-world` project. Yes, I mean it. Type `rm -rf hello-world`. When you run `ls` you should see that `hello-world` no longer
		exists.
1. But that's fine! We can get it back: in your web browser, navigate to your project on Github and copy the URL. Then from within our `code` directory
		we run:

	```sh
	git clone https://github.com/<YOUR USERNAME>/hello-world
	```
1. Now when you run `ls` again, you should see that `hello-world` is back!

**B**) If you are not sure about deleting your project, you can leave it where it is and simply clone it to another location on your computer.

1. Move out of your `hello-world` project: `cd ..`. If you run `pwd` you should see `/Users/<your name>/code`.*
1. We are going to create a temporary place to clone this copy, so that we don't get it confused with the other one we have.
		Make a new temporary directory and move into it: `mkdir temp && cd temp`.**
1. In your web browser, navigate to your project on Github and copy the URL. Then from within our `temp` directory
		we run:

	```sh
	git clone https://github.com/<your username>/hello-world
	```

1. Now when you run `ls`, you should see that `hello-world` is now here.
1. You can run the following to see how many copies you have on your computer and where they are:

	```sh
	find ~ -name hello-world
	# you can google to see how 'find' works
	```

1. And it's that easy! Since we only need one copy, we can delete our new one and the temporary directory we made:

	```sh
	cd ..
	rm -rf temp
	```

_\* Always ensure that you are not cloning repos into other repos. There is sometime a need and a way to do this, but it is far beyond_
_the scope of this tutorial. For now: always check where you are before you clone!_

_\*\* We could have cloned this without creating a temporary directory by simply passing a destination argument to our command:_
_`git clone <URL> hello-world-2`. This would have instructed git to clone the project in a directory named as `hello-world-2`. Without_
_the argument, git will create a directory with the same name as the repo in your Github._


## Part 10: Pulling updates from the cloud

Now that you know you can work on your projects from other computers you are probably wondering: but how do I keep them all in sync?
This is where `git pull` comes in.

As we have discussed in Part 6, the "source of truth" for your projects is always the `upstream` version stored on Github. You should always
therefore aim to ensure your local version is lined up with the remote version before you start adding new commits.*

_\* We will cover what to do if they become misaligned later._

**Steps**:

1. First make sure that any commits you have locally are pushed and that you have no other uncommitted changes.
1. Navigate to your Github project in a web browser.
1. Select your `README.md` from the list of files.
1. At the top right of the file, there are 3 icons: a computer, a pencil, and a trash bin. Click the pencil.
		This will put us into an edit mode. (Yes we can edit files on Github itself! I would not recommend doing this for actual
		code changes, however, as there won't be any syntax highlighting. For markdown it can be helpful, since there is a useful
		`Preview changes` button.)
1. Add a line or two at the bottom of the file. Perhaps noting what you learned in the previous section, or just a test line which
		you can remove later: it is not important.
1. In the `Commit changes` section, a commit message has already been templated for you. You can change this or simply leave it as
		"Update README.md".
1. Click the green `Commit changes` button.
1. Click on the `<> Code` tab at the top left, under your repo name. Underneath, there should be a link to your commits with a number indicating
		how many you have done. Click this, and on the next page you will see your last "Update README.md" is at the top.
1. Now back in your terminal, ensure you are in your project directory with `pwd`.
1. Run `git log`. You should see that your local version is missing that latest commit.
		If your logs are quite long, you will need to hit `q` to exit.
1. Now we can pull down the latest commits simply by running `git pull`. If this fails, it may be because you have local
		uncommitted changes, or because you are not in a git repo.
		Make sure you are in the right place with `pwd` and use `git stash`
		to save any local changes to be dealt with later. If they are not important changes, you can clear them alltogether with `git checkout .`.
1. Now if we run `git log` again, we can see that our local version is now up to date with the remote.
1. If you stashed any changes, you can now get them back with `git stash pop`. Hopefully this will go smoothly and you will be able to
		consider and commit those changes as normal. If you see an error along the lines of `merge conflict`, `git stash` them again and
		you will learn how to resolve those conflicts in a later section.

If you would like to play around more, you can follow the steps in Part 9 Section **B** above, and take it in turns to commit and push something
in one project version and then pull down to another project version. Or if you do have two computers, try pushing and pulling commits between them!

Don't forget: you will have to always push _to_ and pull _from_ the remote version on Github. It is possible to set your upstream to be a version on a personal computer
but that is not in scope for this tutorial, which is about interacting with git via the cloud.

#### Congratulations! You now know about basic project management. Keep going to learn more :).

# Bonus points section! (More complex git flow)
## Part 11: Creating a work in progress branch
Let's add a new feature to our "Hello World!" program: right now it only greets
the world, but it would be cool if it could say hello to specific people.

**Steps**:

1. Move back to your `hello-world` directory: `cd ~/code/hello-world`.
1. Complete the second [Hello World](https://github.com/fouralarmfire/square-one/blob/master/tutorials/hello-world-2.md#hello-world-2) exercise.
1. Run `git status`. It should say that your `hello` file has changed.

	This time, we are not going to commit to `main`. Our "Hello World!" program
	is in an unfinished state: it can't say hello to the world anymore, just to
	whichever arg is passed on the command line.

	When you have a project which others may be using (an app or a computer program)
	it is generally bad form to completely change from the original functionality.

	Imagine if a website you use often were completely to change the way it wanted
	users to interact with it. You would probably be pretty frustrated, since the
	reason you use that site so often is because you like the way it works and know
	how to.

	So, developers have to ensure that when they add new functionality, their users
	experience no disruption in their usual service. If there is a grand plan to
	overhaul the whole thing, it must be done incrementally, usually by inviting
	users to try out the new system without removing the old.

	This means that when we made our `hello` program say hello to just one person
	at a time, we broke that contract with our users. Our new feature is still a
	work in progress so we should not push it to `main` just yet.

	Of course, we could just jump right in and fix it so that both the original
	feature and the new work perfectly together, but then we would learn nothing
	about branch workflow. Branches (along with Pull Requests) are also how coders
	contribute to others' repos.

	Let's pretend that it's the end of the day and we want to go home, or that we
	have to run off to a very important meeting. We still have work to do but we
	want to ensure that what we have done so far is saved somewhere we can't spill
	coffee on it.

	So in order to not pollute the `main` branch, we are going to push to a wip
	(work in progress) branch.

1. Create your new branch by running:

	```sh
	git checkout -b wip-hello-name
	```

	Your computer should return `Switched to a new branch 'wip-hello-name'`.
	`checkout` is the instruction to switch branches, and `-b` tells git to 
	create a new branch if the one you are switching to does not already exist.

1. Now you can add and commit to your wip branch safely away from the main branch:

	```sh
	git add hello
	git commit -m "say hello to person"
	```

1. Push your branch to Github. (We have to be clear which branch we are pushing
	to here, as we haven't told Github to track this one.)

	```sh
	git push origin wip-hello-name
	```

1. If you go to your Github repo in your browser now, you'll see that your `main`
	branch has no record of your 'wip' commit. But, if you click on the `Branch:`
	dropdown button to the left, and switch to your `wip-hello-name` branch, you
	should see that this branch has your new experimental changes.

1. Before we finish this section, you may want to update your README to document
	what you have learned. Don't forget to commit and push afterwards:

	```sh
	git add README.md
	git commit -m "how to create a branch"
	git push origin wip-hello-name
	```

## Part 12: Merging branches

Now let's finish our new feature and merge everything back into `main`.

**Steps**:

1. Complete the third [Hello World](https://github.com/fouralarmfire/square-one/blob/master/tutorials/hello-world-3.md#hello-world-3) exercise.
1. Once your program is able to say hello both to one person and the world, commit
	your changes and push to your 'wip' branch:

	```sh
	git add hello
	git commit -m "say hello to person and world"
	git push origin wip-hello-name
	```

1. Again, if you go to your repo online, you will see that `main` has no record
	of your 2 commits but `wip-hello-name` does.
	Now we need to get the 'wip' work into `main`, for all your users to enjoy.

1. You can also see which commits are on which branch from the command line.
	From your `wip-hello-name` branch, you can run `git log` and see your two
	'wip' commits as well as your original commits which you did on `main`.

	Switch back to the `main` branch with `git checkout main` (you do not need the `-b` this time
	as the branch already exists) and run `git log` again. You should see that `main` only has the commits you made before you
	started working on the new feature.

	To merge those commits into `main`, we need to run:

	```sh
	git merge wip-hello-name
	```

	This may open a merge commit message in the Vim text editor in your terminal. To quit Vim, type `:wq`.
	If unpredictacble things start to happen while in Vim, press the Escape key first, and then `:wq`.
	(You can learn more about how to use the Vim text editor by typing `vimtutor` into a new terminal window.)

	Now if we run `git log` again, we see that `main` now has the commits we
	made on `wip-hello-name`. We can now push our new feature on `main` with `git push`.

1. After `main` is up to date we can delete both our local and remote versions
	of `wip-hello-name`:

	```sh
	git branch -d wip-hello-name # delete local
	git push origin --delete wip-hello-name # delete remote
	```

1. Don't forget to update your README with what you have learned.

## Part 13: Dealing with merge conflicts

What is a merge conflict? Merge conflicts happen when you have two commits which made a change at the exact same line in two separate versions.
When you try to `git pull`, git will need your help to figure out which version you are serious about. Usually even when several
people are working on the same codebase, few are rarely working on the exact same section simultaneously, but it can happen.

We are going to deliberately create a merge conflict so we can understand how to fix it.

**Steps**:

1. First make sure your local is up to date with your remote: ensure any commits are pushed and that you have pulled any changes
	made remotely or in another version.
1. In your web browser, navigate to your `README.md` and click on the edit button (see the first few steps of Part 10 if you need a refresh).
1. Edit the first line of the file. It doesn't matter what you type, just make it different.
1. Click the green `Commit changes` button.
1. Do **NOT** pull the changes locally.
1. Now, open your local version of your `README.md` in a text editor and edit the exact same first line as you did in the browser.
	This time you may want to make a change you would be happy to keep (we will have to keep one of these versions, this may as well
	be it).
1. Add and commit your changes.
1. And now we can run `git pull` in the terminal.
	The last few lines of git's output should be something like:

	```
	CONFLICT (content): Merge conflict in README.md
	Automatic merge failed; fix conflicts and then commit the result.
	```
1. Run `git status`. The output is telling us that the local and remote versions have diverged, and that each
	has one commit which the other doesn't. It is telling us that both have modified the `README.md` file.
	We have to fix the conflicts and commit the changes we _do_ want.
1. Open your `README.md` in a text editor. It's probably not how you left it in either version, and it may be a little confusing to read,
	but with practice it is possible to understand:
	```
	<<<<<<< HEAD
	the changes you made locally are highlighted here, they are the HEAD (ie the latest point) at which your local version sits
	=======
	the changes you made elsewhere are highlighted here
	>>>>>>> somenonsenseshaheredontworryaboutit
	```
	Now you have to decide which commit you actually want to keep. Simply delete the line(s) you don't want, and also any line
	with `<<<<<<<` or `=======`.
1. Once your `README.md` is back in a readable state, save the file, run `git add` and then `git commit` without any message.
		For this commit, as it did for the branch merge in the previous part, git will open vim. Type `:wq` and then enter to exit vim.
1. If you try to pull now, it should say `Already up to date` and you should now be safely able to push.

This is a very simple example of a merge conflict dealing with just one line in one file.
Huge conflicts can happen with multiple files and many lines, and they can be quite confusing
to sort out, but by following the pattern above they are manageable. Always remember to read the git message:
more often than not, git will tell you what to do.

&nbsp;

<sup>_Mistakes? Corrections? Suggestions?_ <a href="https://github.com/cbctl/cbctl.github.io/tree/master/tutorials/git-and-github.md"><svg class="svg-icon"><use xlink:href="/assets/minima-social-icons.svg#github"></use></svg></a></sup>

<sup>_Is something unclear? Do you need help?_ <a href="https://github.com/cbctl/cbctl.github.io/issues/new"><svg class="svg-icon"><use xlink:href="/assets/minima-social-icons.svg#github"></use></svg></a></sup>

### This tutorial is a work in progress. Refresh regularly for updates.
