---
title: Binary Multiplication
...

This is the first of two assignments in which you will write binary code for the simple machine you created a simulator for [in lab](lab03-simulator.html). It is much simpler than the second one, and will take much less time. Both assume that you have understood that lab's content well.

# The instructions

+----------+-------------------------------------------------------------------+
|`icode`   |Behavior                                                           |
+:========:+:==================================================================+
|   0      |`rA = rB`                                                          |
+----------+-------------------------------------------------------------------+
|   1      |`rA += rB`                                                         |
+----------+-------------------------------------------------------------------+
|   2      |`rA &= rB`                                                         |
+----------+-------------------------------------------------------------------+
|   3      |`rA =` read from memory at address `rB`                            |
+----------+-------------------------------------------------------------------+
|   4      |write `rA` to memory at address `rB`                               |
+----------+-------------------------------------------------------------------+
|   5      |do different things for different values of `b`:                   |
|          |                                                                   |
|          |+-----+-----------------------------------------------------------+|
|          ||`b`  |action                                                     ||
|          |+:===:+:==========================================================+|
|          || 0   |`rA = ~rA`                                                 ||
|          |+-----+-----------------------------------------------------------+|
|          || 1   |`rA = -rA`                                                 ||
|          |+-----+-----------------------------------------------------------+|
|          || 2   |`rA = !rA`                                                 ||
|          |+-----+-----------------------------------------------------------+|
|          || 3   |`rA = pc`                                                  ||
|          |+-----+-----------------------------------------------------------+|
+----------+-------------------------------------------------------------------+
|   6      |do different things for different values of `b`:                   |
|          |                                                                   |
|          |+-----+-----------------------------------------------------------+|
|          ||`b`  |action                                                     ||
|          |+:===:+:==========================================================+|
|          || 0   |`rA =` read from memory at `pc + 1`                        ||
|          |+-----+-----------------------------------------------------------+|
|          || 1   |`rA +=` read from memory at `pc + 1`                       ||
|          |+-----+-----------------------------------------------------------+|
|          || 2   |`rA &=` read from memory at `pc + 1`                       ||
|          |+-----+-----------------------------------------------------------+|
|          || 3   |`rA =` read from memory at the address stored at `pc + 1`  ||
|          |+-----+-----------------------------------------------------------+|
|          |                                                                   |
|          |In all 4 cases, increase `pc` by 2, not 1, at the end of this      |
|          |instruction                                                        |
+----------+-------------------------------------------------------------------+
|   7      |Compare `rA` (as an 8-bit 2's-complement number) to `0`;           |
|          |if `rA <= 0`, set `pc = rB`                                        |
|          |otherwise, increment `pc` like normal.                             |
+----------+-------------------------------------------------------------------+

# Running programs

You should create two files

1. One you work with, that has comments and notes to keep you sane. Call this anything you like.

1. One you run and submit, which contains nothing by hex bytes separated by white space. You'll submit this as a file named `mult.binary`

To test your code, do one of

    python3 sim_base.py mult.binary

or

    java SimBase mult.binary

or going to [our online simulator](files/toy-isa-sim.html) and click the file upload button at the top of the page to load your `mult.binary` into the simulator's memory.

# Your task

Your code should

1. load the values in memory at addresses 0x01 and 0x03 into registers
2. compute the product of those values (i.e., multiply)
3. store the product at address 0xA0
4. halt once it is done

You should ignore overflow, so since 0xC9 × 0x23 = 0x1B7B, the answer should be 7B. This is likely to happen automatically without your explicit planning for it.
You may assume that neither multiplicand will be negative, but either or both may be zero.

Thus, if `mult.binary` begins `__ 09 __ 0A` then when it is finished it should have `5A` in address 0xA0;
if `mult.binary` begins `__ 49 __ 23` then when it is finished it should have `7B` in address 0xA0.
We should be able to change the first second and fourth bytes of your program to do other multiplications too.

# Hints, tips, and suggestions

## How to multiply

You may be familiar with fancier ways, but a simple definition of multiplication that will result in reasonable-to-write code is

Definition
:   *x* × *y* means the sum of *y* *x*'s; that is, *x* + *x* + *x* + ... + *x*, where the number of *x*s is *y*.

You *definitely* want to make sure you can write working code for this in some language you know well before trying to convert that code into binary.

## How to write binary

We suggest following these steps, carefully, saving the result of each in a file so you can go back and fix them if they were wrong:

1. Write pseudocode that does the desired task
2. Convert any `for` loops to `while` loops with explicit counters
3. Change any `if` or `while` guards to the form `something <= 0`
    - `a <= b` becomes `a-b <= 0`
    - `a < b` becomes `a+1 <= b` becomes `a+1-b <= 0`
    - `a >= b` becomes `0 >= b-a` becomes `b-a <= 0`
    - `a > b` becomes `0 > b-a` becomes `b+1-a <= 0`
    - `a == b` becomes `a-b == 0` becomes `!(a-b) == 1` becomes `!!(a-b) <= 0`
    - `a != b` becomes `a-b != 0` becomes `!(a-b) == 0` becomes `!(a-b) <= 0`
4. Add more variables to split multi-operation lines into a series of single-operation lines
5. Add more operations to convert ones not in the instruction set into ones in the instruction set
6. Change each loop into a pair of instructions, opening with "`spot1` = `pc`" and closing with "if ..., goto `spot1`"
7. Count the number of variables needed
    - If^[depending on how you write your original code, this is possible for this task …] it is ≤ 4, skip to step 10
    - else^[… some solutions are in this case instead.], continue with next step
8. Pick a memory address for each variable. Make these big enough your code is unlikely to get that big; for example, you might pick `0x80` though `0x80` + number of variables
9. Convert each statement that uses variables into
    a. register ← load variable's memory
    b. original statement
    c. store variable's memory ← register
10. translate each instruction into numeric (`icode`, `a`, `b`) triples, possibly followed by a `M[pc+1]` immediate value
11. turn (`icode`, `a`, `b`) into hex
12. Write all the hex into `mult.binary`

Debugging binary is hard. That's part of why we don't generally write code in binary. If you get stuck, you should probably try pulling just the part you are stuck on separate from the rest and test it until it works, then put it back in the main solution.


# Submit

Submit via the submission site, linked from the top of every course page and also available as <https://kytos.cs.virginia.edu/coa1/>
