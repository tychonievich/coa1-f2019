---
title: Bit fiddling homework
...


# Overview

This homework will give you a chance to practice using binary and bit-wise operators.
You'll likely find [Booleans §4](bool.html) a useful reference.

# Task

Visit <https://kytos.cs.virginia.edu/coa1/pa01.php>
and complete at least 80% of the problems there.
The text boxes want lightweight code using just operators and assignments, like

````c
x = 0x20
y = b + x
````

The goal is to end up with one variable having a particular value,
based on other variables that are provided with new values in each test case.
Do not add conditionals, loops, subroutines, etc.

## The Tasks

In case the server is down, the tasks in question are:

subtract
:   Given `x` and `y`, set `z` to `x - y` without using `-` or multi-bit constants.

    For full credit, use ≤ 10 operations from {`!`, `~`, `+`, `*`, `%`, `/`, `<<`, `>>`, `&`, `^`, `|`}.

bottom
:   Given `b`, set the low-order `b` bits of `x` to 1; the others to 0. For example, if `b` is 3, `x` should be 7. Pay special attention to the edge cases: if `b` is 32 `x` should be &minus;1; if `b` is 0 `x` should be 0. Do not use `-` in your solution.

    For full credit, use ≤ 40 operations from {`!`, `~`, `+`, `*`, `<<`, `>>`, `&`, `^`, `|`}.

anybit
:   Given `x`, set `y` to `1` if any bit in `x` is `1`; set `y` to `0` if `x` is all `0`s.

    For full credit, use ≤ 40 operations from {`~`, `+`, `-`, `<<`, `>>`, `&`, `^`, `|`}.

fiveeighths
:   Given `x`, set `y` to be 5/8 of `x` (rounded toward zero). This should work for both positive and negative numbers, even if neither `5*x` nor `x/8` can be properly represented in 32 bits, but does not need to work for 0x80000000.

    For full credit, use ≤ 20 operations from {`!`, `~`, `+`, `*`, `%`, `/`, `<<`, `>>`, `&`, `^`, `|`}.

bitcount
:   Given `x`, set `y` to the number of bits in `x` that are `1`.

    For full credit, use ≤ 40 operations from {`!`, `~`, `+`, `-`, `<<`, `>>`, `&`, `^`, `|`}.



# Collaboration

You may work with other students in this class on this assignment, but only in the following two ways:

1. You worked together from the beginning, solving the problem as a team, with each person contributing.
    
    Each teammate should cite this in each problem with a C-style comment at the top of each solution
    and also cite the originator of any single-person contributions where they appear, like
    
    ````c
    // Part of a team with mst3k and lat7h
    x = -y
    w = -x // lat7h came up with this line
    z = x + y
    ````
    

2. You helped someone with a task you'd already finished, helping them think through their incorrect solution and not giving them or trying to lead them to your solution.

    The helper should acknowledge they did this by returning to their previously-submitted solutions
    and re-submitting them with an added comment at the top, like
    
    ````c
    // I helped tj1a
    x = -y
    w = -x
    ````
    
    The helpee should acknowledge they got this by adding a comment at the top, like
    
    ````c
    // tj1a helped me
    x = -y
    w = -x
    ````
    
In all cases, include computing IDs in your citations to streamline our automated tools that assist with collaboration checking.
