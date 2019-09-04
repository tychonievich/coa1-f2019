---
title: Information Theory Lab
...

# This lab has three purposes

1. Resolve any concerns you may have from [Lab 00](lab00-ssh-ed.html)
2. Introduce the git tool
3. Have you explore Shannon's information theory

Please work through these in order, as the first is most important and not quite finishing the last shouldn't be too much of a problem.

# Lab 00 review

You should be able to

1. Navigate in a shell, into and out of directories, listing what is in a directory and seeing its contents
1. Edit files from the shell
    - we don't care which CLI editor you use: nano, vim, emacs, any is fine.
1. Access `portal.cs.virginia.edu` via `ssh`
    - If you want to use `ssh` without the need to re-type your password each time, see [our ssh help sheet](help-ssh.html#using-key-pairs-instead-of-passwords)
    - If your password isn't working, try <https://www.cs.virginia.edu/PasswordReset>; if that still doesn't work, contact your professor via email
1. recognize 
    - `.` as meaning "the current directory"
    - `..` as meaning "the directory containing the current directory"
    - `~` as meaning "my home directory" -- i.e., `/u/lab/mst3k`

If you are unsure of any part of this, please talk to a TA in lab. You take priority over people working on later parts of this lab.

## One more step: changing permissions

### Overview of directory permissions

Each directory has three permissions, called "read", "write", and "execute".

read
:   Turns on the lights, so you can see what's inside.
    Without read permission, `ls` won't work in the directory.

execute
:   Unlocks the doors, so you can move through it.
    Without execute permission, `cd` won't work into the directory
    and nothing will work with a path that includes the directory in the middle.

write
:   Lets you change what's in the directory.
    Without write permission, you can neither create nor remove
    files and directories inside a directory.

You can change permissions using the `chmod` command.
It has multiple ways to be used, but a few simple examples are

````bash
chmod a+r foo   # all users can read foo
chmod a-w foo   # no user can write foo
chmod u+w foo   # the owning user can write foo
````

### Overview of file permissions

Directories can contain other directories, and also can contain files.
Files, like directories, have names and permissions but cannot be entered with `cd`.
The permissions also have different meaning than with directories:

read
:   Lets you see the contents of the file.

execute
:   Lets you (try to) treat the file like a program.

write
:   Lets you change the contents of the file.

### Overview of groups

Every user belongs to one ore more groups, and every file or directory has an owning user and an owning group.
Permissions are specified as read/write/execute for the user, group, and others.

You can find out your user name with `whoami` and your group memberships with `groups`.

### Do this!

You should set up your home directory so only you can access it, not other people in your group nor strangers not in your group:

After `ssh`ing into the CS server (i.e., `ssh mst3k@portal.cs.virginia.edu`{.bash}), run the following commands:

````bash
cd              # go home
chmod g-rwx .   # remove group-access to read, write, and execute this directory
chmod o-rwx .   # remove other-access to read, write, and execute this directory
````


# `git` introduction

We'll use git in this class, and many other classes that follow.
Git is a versatile tool that does many useful things; among them are

- It stores all versions of your files so you can recover from mistakes
- It lets you move files between machines easily
- It lets multiple people collaborate on a project with minimal coordination
- It lets you set up "hooks" to automatically run when things change

## Broad overview

- Several computers will be involved
- Each will have a complete copy of the history of every file
- On a single computer,
    - Each file has two places it is stored: the working directory (which is where you'll work on it) and a hidden git-managed copy (which is where git will track versions, communicate with other machines etc)
    - When you edit a file, git considers it either "untracked" or "modified"
        - an "untracked" file has no copy in the git-managed space
        - a "modified" file has been changed from the last git-managed version
    - "adding" a file tells git to care about your changes to it
    - "committing" a file tells git top update the git-manages copy to match your version. You can only commit added files.
- Working between computers,
    - you can "push", saying "remote computer, you should know about what I've done lately."
    - you can "pull", saying "remote computer, what have you done lately?"
    - In general you have to both pull and push if you want your copy nad the remote copy to become the same.

## Most common case

### Creating a project

You'll only need to do this once in this course, so we'll only give a little explanation.

1.  Create the git-managed project on the server, `portal.cs.virginia.edu`.
    Use your user name, not `mst3k`, and any name you want (we assume `coa1-code` but you can change that)

    ````bash
    ssh mst3k@portal.cs.virginia.edu
    mkdir coa1-code.git
    cd coa1-code.git
    git init --bare
    exit
    ````

1.  Create your local working copy of that project
    
    ````bash
    git clone mst3k@portal.cs.virginia.edu:coa1-code.git
    cd coa1-code
    ````

1.  Create a working copy on the server
    
    ````bash
    ssh mst3k@portal.cs.virginia.edu
    git clone mst3k@portal.cs.virginia.edu:coa1-code.git
    exit
    ````

We may have you use projects we've made for you later in the semester, which uses step 2 of the above.

### Laptop → git → CS server

We expect the most common case will be you've created or modified a file on your laptop and want to try running it on the server.
Let's go through this step by step, assuming you start in Terminal/PowerShell in the directory of your project on your laptop

`git add file1 file2 ...`{.bash}
:   Tell `git` which files you want to have sent.
    If you want *everything* in the current directory sent, you can use `git add .`{.bash}.
    
    If you created no new files, you can skip this step as described below.

`git commit -m "I changed a few files"`{.bash}
:   Tell `git` to update it's internal copy of the files you've added and label this change "I changed a few files".
    Using good descriptive labels becomes more important as project teams grow.
    
    You can add and commit in one step if you have only modified (not added) files by using `git commit -a -m "fixed typos"`{.bash}

`git pull`{.bash}
:   Now that git knows what you've done on your laptop, have it check to see if some other computer has also made changes that you'll need to merge.
    We don't expect that to happen very often this semester, but you should get into the practice of *always* `git pull`{.bash} before `git push`{.bash}.

`git push`{.bash}
:   As git to send your changes to the master repository on the remote server

`ssh mst3k@portal.cs.virginia.edu`{.bash}
:   Move over to the server

`cd coa1-code`{.bash}
:   Enter the server's project directory

`git pull`{.bash}
:   Grab the code you just pushed from your laptop into the server's copy

Compile and run your program
:   however is appropriate for what you are doing

## Task for this lab

1. Create a git project with a copy on your laptop and on the server, as described under "[Creating a project](#creating-a-project)" above

On your laptop, in the directory of this new project, create a file named `lab01` that contains

    I wrote this on my laptop! Hooray!

then use commands from "[Laptop → git → CS server](#laptop-git-cs-server)" above to get it into git and onto the server's version of the project.
Then show the TAs you did so.
One way to show this is to `ssh` into the server, go to your project directory, and type the following

```bash
ls -l
pwd
ls -ld ~
git status
cat lab01
```

... then call over a TA to show them the results.

## Tutorials

There are several tutorials you may find useful. Don't feel that you have to do all of these (unless you want to and/or you need to still learn it), but doing at least one all the way through may help you feel more confident.
You should probably do them on your own time, though, not during lab time.

1. <https://git-scm.com/docs/gittutorial>
1. <https://try.github.io/>
1. <https://learngitbranching.js.org/>
1. <https://rogerdudler.github.io/git-guide/>
1. <https://guides.github.com/introduction/flow/>
1. <https://guides.github.com/activities/hello-world/>

# Information Theory

## Introduction

Claude Shannon founded the field of information theory.
A core fact in information theory is that there is a basic unit of information,
called a "bit^[a portmanteau of "binary digit"]" or a "bit of entropy."
Roughly speaking, a "bit" is an amount of information that is about as surprising as the result of a single coin flip.
In the sentence "Hello, my name is Claude" the word "is" has low entropy; most of the time someone says "Hello, my name" the next word they say is "is," so adding that word provides very little more information.
On the other hand, the word "Claude" in thart same sentence has many bits of entropy; a huge number of words could reasonably follow "Hello, my name is" and any given one we pick is quite surprising.

In computers, we rarely encode things anywhere near as efficiently as its bits of entropy suggest.
For example, most common encodings of strings use 8 bits per character.
In this lab, you'll replicate an experiment Claude Shannon published in 1950^[Shannon, Cluade E. (1950), "Prediction and Entropy of Printed English", *Bell Systems Technical Journal* (3) pp. 50--64.] to see just how inefficient that encoding is.


## Preparation

First, you'll write a program in either Python or Java.
Then, you'll use that program to perform an experiment and reflect on the results.

You may either work alone or with a buddy in this lab.
Buddy programming is where two people work side-by-side,
each creating similar programs while talking together to help one another out.
In well-running buddy programming each buddy is speaking about equally,
describing what they are writing next or how they are testing what they have written.
Buddy programming usually results in similarly-designed but non-identical programs.

If you use a buddy, you should sit next to your buddy and use the same programming language they use.

## Create the program

Your program should do the following:

1. Read a text file into a string in memory.
    You should be able to specify different file names each time you run the program.

2. Repeatedly

    a. Pick a random index in the middle of the string
    b. Display to the user the 50 characters preceding that index (in such a way that they can tell if what you displayed ended in a space character or not)
    c. Have the user type a single character
    d. Record if that typing was correct

3. After some fixed number of iterations (20 might make sense), display
    
    - The ratio of correct guesses (e.g., "`You got 14 out of 20 guesses correct!`")
    - The estimated bits of entropy per letter of the text, which is
        log~2~(*g* ÷ *r*) where *g* is the total number of guesses made
        and *r* is the number that were correct
        (e.g., 0.5145731728297582 for 14 of 20 correct).

## What is the entropy of...

Once your program seems to be working, try it on a few different texts.
For example, you might try

- [tarzan.txt](files/tarzan.txt) (500KB) -- the original Tarzan book by Edgar Rice Burroughs.
- [pi.txt](files/pi.txt) (1MB) -- the first million digits of pi.
- [_pydecimal.py](files/_pydecimal.py) (229KB)-- a large file from the Python standard library.
- [diff_patch_match.java](files/diff_patch_match.java) (89KB) -- a large file from the open source Java project [diff-match-patch](https://github.com/google/diff-match-patch).

Add a comment to the top of your code that includes at least the following:

- Who your buddy was, if any
- What files you tested (if other than the above, with their full URL or a description of what they contained) and what the results were for each
- An additional experiment you did and how it came out. For example, you might try to answer
    - is language X more or less entropic than language Y?
    - does it matter how many characters you display as context for their guess?
    - is the answer different if you display the characters after, not before, the one they guess?
    - if you re-run the test on the same file repeatedly, how consistent are the answers?
    - if you compress the file (e.g., into a .zip file or the like), how much smaller does it get? How (and why) is this related to its bits of entropy?

## Check-off

Show a TA your working code.
They may ask you some questions about how your code works and what you think of the results.
For most students, this should happen in lab;
if you have completed the lab exercise before lab occurs, you are welcome to do it in their office hours.
