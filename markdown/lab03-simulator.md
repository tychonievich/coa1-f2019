---
title: Simulator
...

# Introduction

As discussed in class, register-transfer level design can be relatively simply described as

1. clock signal has a rising edge
2. register outputs change
3. logic works, which can be thought of as arbitrary acyclic code that assigns each variable only once
4. logic results in register inputs changing
5. repeat

In this lab, and the subsequent programming assignment, you will expand on these basic pieces to build your own machine simulator and write binary code for it.

# Getting the simulator skeleton

We've written a basic simulator in [python (sim_base.py)](files/sim_base.py) and [java (SimBase.java)](files/SimBase.java).
This works from a command-line interface, expecting memory byte values (either directly or in a file) as a command-line argument:

````sh
# runs SimBase in java with 8 bytes of memory set
javac SimBase.java
java SimBase 01 23 45 67 89 ab cd ef
````

````sh
# runs SimBase in python with 8 bytes of memory set
python3 sim_base.py 01 23 45 67 89 ab cd ef
````

````sh
# runs SimBase using the contents of memory.txt to set memory
python3 sim_base.py memory.txt
java SimBase memory.txt
````

Memory contents must be specified in hexadecimal bytes, separated by whitespace.

# Write the simulator functionality

Each file begins with a function or method named `execute` which is given two arguments (the current instruction in `ir` and the PC of this instruction in `oldPC`) and returns one value (the `pc` to execute next).
The skeleton code just returns `oldPC + 1`.

Each file also has two global values you can access: `R`, an array of 4 register values, and `M`, an array of 256 memory values.

We also provide a helper function for getting a range of bits from a number.

{.exercise ...}
Add code to `execute` (do not edit other parts of the file) to do the following:

Separate the instruction into parts
:   Treat the instruction as having four parts:

    --------------------------------------------------------------------------------
    bits    name        meaning
    ------  ----------  ------------------------------------------------------------
    7       reserved    If set, an invalid instruction.
                        Do not do work or advance the PC if this bit is 1.

    \[4, 7) `icode`     Specifies what action to take

    \[2, 4) `a`         The index of a register

    \[0, 2) `b`         The index of another register, or details about icode
    --------------------------------------------------------------------------------

Execute the instruction
:   (You will probably need to be familiar with the [Language nuances](#language-nuances) of your implementation language of choice before starting on this section.)

    Change `execute` to do different things for different `icode`s, as follows:


    - If `reserved` is 1, set the next PC to the current PC instead of advancing it and do nothing else.

    - Otherwise, see the following table, where `rA` means "the value stored in register number `a`" and `rB` means "the value stored in register number `b`."
        Unless a different value of the `pc` is specified in the table, also add one to the `pc` for each instruction.


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

Test your simulator
:   See [Example programs](#example-programs) below for suggestions.

{/}

## Language nuances

Python's syntax for `!x`{.c} is `not x`{.python} instead.

Java treats bytes (like `R[i]`{.java}) as signed integers, not unsigned.
That means they are not good indices (e.g., `M[R[i]]`{.java} might throw an exception if `R[i]`{.java} is negative).
However, `R[i] & 0xFF`{.java} treats it as unsigned instead,
so `M[R[i] & 0xFF]`{.java} should work.

Python treats bytes as unsigned, so `R[i] <= 0`{.python} is always true.
It can be re-written to work correctly as `R[i] == 0 or R[i] >= 0x80`{.python}


## Example programs

Halt
:   The code

        00 00 00 80

    should advance to `pc` 3 and then stay there.
    
    Try finding other code that will also do that.

Move
:   Consider the following code
    
    Bytes   Activity
    ------  -------------
    68 23   move 0x23 into register 2
    06      move from register 2 to register 1
    60 20   move 0x20 into register 0
    44      move from register 1 to memory at address from register 0 (i.e., move 0x23 to address 0x20)
    80      halt
    
    If your simulator works, then `68 23 06 60 20 44 80` should end with registers being `[20, 23, 23, 00]` and a 0x23 in memory at address 0x20.
    
    Try adding code that will read from memory into register 3.
    
Math
:   Consider the following code

    Bytes       Activity
    ------      -----------------
    68 10       move 0x10 into R~2~
    32          move value from address R~2~ into R~0~
    68 11       move 0x11 into R~2~
    36          move value from address R~2~ into R~1~
    14          R~1~ += R~0~
    68 12       move 0x12 into R~2~
    46          move R~1~ into address R~2~
    80          halt
    five 00s    (padding)
    23 A1       some numbers to add
    
    If your simulator works, you should end up with 0xC4 (i.e., 0x23 + 0xA1) in address 0x12.
    
    Try adding code that can sum more numbers from memory, not just those two.

Conditional jump
:   The following should not jump, so three steps should end up with the PC at address 5, not 20:
    
    pseudocode                  parts           bytes
    --------------------------  ------------    -------------
    R~0~ = 10                   (6, 0, 0) 10    60 0A
    R~1~ = 20                   (6, 1, 0) 20    64 14
    if R~0~ <= 0, jump to R~1~  (7, 0, 1)       71

    The following should jump, so three steps should end with the PC at address 20, not 5:
        
    pseudocode                  parts           bytes
    --------------------------  ------------    -------------
    R~0~ = −10                  (6, 0, 0) −10   60 F6
    R~1~ = 20                   (6, 1, 0) 20    64 14
    if R~0~ <= 0, jump to R~1~  (7, 0, 1)       71

Read from immediate address
:  The following should load 20 into R~1~
    
    pseudocode                  parts           bytes
    --------------------------  ------------    -------------
    R~2~ = 10                   (6, 2, 0) 10    68 0A
    R~3~ = 20                   (6, 3, 0) 20    6C 14
    write R~3~ to address R~2~  (4, 3, 2)       4E
    read address 10 into R~1~   (6, 1, 3) 10    67 0A
    
    You could also do this without using instruction 4
    by setting enough memory that there was already data in address 10:
    
    pseudocode                      parts           bytes
    ----------------------------    ------------    ------------------------
    read address 10 into R~1~       (6, 1, 3) 10    67 0A
    intialize address 10 with 20    0s, then 20     00 00 00 00 00 00 00 14
    
## More, less-explained examples

icode = 0
:   First load a value into R~1~ then set R~2~ to equal R~1~: `64 14` `09`

icode = 1
:   First load a value into R~1~ and R~2~ then add them together: `64 14` `68 20` `19`

icode = 2
:   First load a value into R~1~ and R~2~ then and them together: `64 14` `68 20` `29`

icode = 3
:   First load a value into R~1~ and then load that memory address into R~2~: `64 02` `39`

icode = 4
:   First load a value into R~1~ and R~2~ and then store R~1~ into memory at R~2~: `64 02` `68 20` `46`

icode = 5
:   First load a value into R~1~ and then do something to it: 
    
    - flip all bits: `64 89` `54` (result: `76`)
    - perform `!`: `64 89` `55` (result: `00`; try also `55` by itself to result in `01`)
    - negate: `64 89` `56` (result: `77`)
    - replace with the PC: `64 89` `57`  (result: `02`)

icode = 6
:   First load a value into R~1~ and then use an immediate: 

    - replace with immediate: `64 14` `64 20` (result: `20`)
    - add immediate: `64 14` `65 20` (result: `34`)
    - and immediate: `64 14` `66 24` (result: `04`)
    - replace with memory contents at immediate: `64 14` `67 02` (result: `67`)

icode = 7
:   First load an address into R~1~, a value into R~2~, and then conditionally jump: 

    - jumps: `64 14` `68 ff` `79`
    - jumps: `64 14` `68 80` `79`
    - jumps: `64 14` `68 00` `79`
    - does not jump: `64 14` `68 01` `79`
    - does not jump: `64 14` `68 7f` `79`

# Check-off

To check-off this lab, show a TA your simulator and the memory contents that do the tasks listed above.

