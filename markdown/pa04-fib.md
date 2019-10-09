---
title: Binary Fibonacci
...


This is the second of two assignments in which you will write binary code for the simple machine you created a simulator for [in lab](lab03-simulator.html). It is more involved than the first one, and will take more time. Both assume that you have understood that lab's content well.

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

1. One you run and submit, which contains nothing by hex bytes separated by white space. You'll submit this as a file named `fib.binary`

To test your code, do one of

    python3 sim_base.py fib.binary

or

    java SimBase fib.binary

or going to [our online simulator](files/toy-isa-sim.html) and click the file upload button at the top of the page to load your `fib.binary` into the simulator's memory.

# Your Task

Create a binary program (i.e., a file containing hex bytes)
that runs in this language; name the file `fib.binary`.

When run in a simulator (yours or [ours](files/toy-isa-sim.html)), `fib.binary`
should change the contents of memory for all addresses `i` ≥ C0~16~,
placing in address `i` the `i-0xC0`th Fibonacci number (modulo 256, since these are bytes).

Once `0xC0` through `0xFF` are set, halt by running an instruction with the `reserved` bit set.

The file `fib.binary` itself must not contain more than C0~16~ (192~10~) hexadecimal bytes.

It should be the case that running your simulator on `fib.binary` for many cycles should result in output ending with the following:


    0xc0-cf: 01 01 02 03 05 08 0d 15 22 37 59 90 e9 79 62 db
    0xd0-df: 3d 18 55 6d c2 2f f1 20 11 31 42 73 b5 28 dd 05
    0xe0-ef: e2 e7 c9 b0 79 29 a2 cb 6d 38 a5 dd 82 5f e1 40
    0xf0-ff: 21 61 82 e3 65 48 ad f5 a2 97 39 d0 09 d9 e2 bb


# Hints, tips, and suggestions

## How to compute Fibonacci numbers

1. Keep track of two numbers, current and previous. Start them both off at 1.
2. Let next be the sum of current and previous.
3. Rename current → previous, next → current (in that order)
4. Repeat

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
    - else^[… but most solutions are in this case instead.], continue with next step
8. Pick a memory address for each variable. Make these big enough your code is unlikely to get that big; for example, you might pick `0x80` though `0x80` + number of variables
9. Convert each statement that uses variables into
    a. register ← load variable's memory
    b. original statement
    c. store variable's memory ← register
10. translate each instruction into numeric (`icode`, `a`, `b`) triples, possibly followed by a `M[pc+1]` immediate value
11. turn (`icode`, `a`, `b`) into hex
12. Write all the hex into `mult.binary`

Debugging binary is hard. That's part of why we don't generally write code in binary. If you get stuck, you should probably try pulling just the part you are stuck on separate from the rest and test it until it works, then put it back in the main solution.
