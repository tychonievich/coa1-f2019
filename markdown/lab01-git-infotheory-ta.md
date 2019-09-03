---
title: Information Theory Lab -- TA guidelines
...

We're tracking lab completion on a google sheet: <no link yet>

Prioritize questions on lab 00 (shell, ssh, ed) over other questions

If people don't finish the info theory part, that's fine; but do track how many don't so that I can update expectations for future labs.

I recommend walking them through the `chmod` bit as a class using the projector.
If it works, they should be able to run `ls -ld ~` and see something like

    drwx------ 3 mst3k lab_fall2019 7 Jan  8  2018 /u/lab/mst3k

... where the `------` is the most important part (no one else can access their directory).


If they do the git part properly,

when they run  they'll see something like
-------------- -----------------------------------------------------------
`ls -l`        `-rw-r--r-- 1 mst3k lab_fall2019  35 Sep  4 17:19 lab00`
`pwd`          `/u/lab/mst3k/coa1-code`
`ls -ld ~`     `drwx------ 3 mst3k lab_fall2019 7 Jan  8  2018 /u/lab/mst3k`
`git status`   `On branch master` <br> `Your branch is up to date with 'origin/master'.` <br> either `nothing to commit, working tree clean` or a list of untracked files not including `lab00`.
`cat lab01`     `I wrote this on my laptop! Hooray!`


When they get to checking off the information theory bit, be kind; anything that seems reasonable is OK.
