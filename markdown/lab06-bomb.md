---
title: Bomb lab
...

A Mad Programmer got really mad and created a slew of "binary bombs".
Each binary **bomb** is a program, running a sequence of phases.
Each **phase** expects you to type a particular string.
If you type the correct string, then the phase is defused and the bomb proceeds to the next phase.
Otherwise, the bomb **explodes** by printing "BOOM!!!", **telling us it did so**,
and then terminating.

# Work together

In lab, we strongly encourage you to work with one another.
Reading binary is much more fun and effective with someone else to talk to.

You should *not* work together on phase 2, as that is a PA.

# Grading

You'll use the same bomb for this lab and for [the following PA](pa06-bomb.html).

For lab, you need to *either* (a) have a TA record that you were part of a team that defused phase 1 *or* (b) defuse phase 1 on your bomb.

For the PA, you'll need to defuse additional phases on your own.

Each time your bomb explodes it notifies the bomblab server. If we're notified of your bomb exploding 20 times we’ll start removing points.


# How to proceed

1. On a Linux machine, download[^curl] a [binary bomb](http://kytos.cs.virginia.edu:15215/)
    - for credit, you must use your lower-case computing ID

    > Due to changes needed to fix several errors with the bomb server, bombs downloaded before Wednesday, 16 October 2019 at 6:00pm will not be graded for the PA. Please download a new bomb if yours is older than that.

2. Extract the bomb using `tar -xvf bomb#.tar`{.bash} where `#` is your bomb number.
3. `cd bomb#` (again, where `#` is your bomb number).
4. Read the `README`
5. You are welcome to look at `bomb.c` -- it isn't very interesting, though
6. Do whatever you need to to understand what the bomb is doing
7. Only run the bomb `./bomb` once you are confident you can defuse a phase (or at least avoid an explosion)
8. Once you pass a phase visit [the scoreboard](http://kytos.cs.virginia.edu:15215/scoreboard) to verify that we saw your success.

[^curl]: If you want, you can download on portal with the following two lines:
    ````bash
    curl "http://kytos.cs.virginia.edu:15215/?username=$USER&usermail=$USER@virginia.edu&submit=Submit" > bomb.tar
    tar xvf bomb.tar
    ````

## Hints

If you run your bomb with a command line argument, for example, `./bomb psol.txt`, then it will read the input lines from `psol.txt` until it reaches EOF (end of file), and then switch over to the command line. This will keep you from having re-type solutions.

Because you want to avoid explosions, you'll want to set a breakpoint *before* you run the program so that you can stop the program before it gets to a the function that does the exploding.

You might find it useful to run, `objdump --syms bomb` to get a list of all symbols in the bomb file, including all function names, as a starting point on where you want your breakpoint.
