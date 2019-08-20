---
title: Designing a processor
author: Luther Tychonievich
license: <a href="https://creativecommons.org/licenses/by-nc-sa/4.0/">Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0)</a>
...

Most machines, from the simplest hammers to the most sophisticated appliances,

- have a finite set of possible operations (e.g., a car can accelerate, turn, and decelerate)
- use control input to determine the operation to use (e.g., the steering wheel and pedals provide input to a car)
- interact with data to determine the results of the operation (e.g., the pitch of the roadway)
- result in modification of some part of their environment (e.g., the position and velocity of the car)

A general-purpose computing machine is a special machine where the control, data, and modified environment are all the same.
This allows one to have the computer modify its own instructions,
expanding the capability of the finite set of core operations to perform arbitrarily complex work.
General-purpose computing machines have become so pervasive that they are now commonly called simply "computers," a term formerly reserved for a human occupation.

# A brief history of computers

The first known general-purpose computer to be designed was Charles Babbage's Analytical Engine,
designed and partially constructed in the 1830s.
Although Babbage designed it, it was Ada King-Noel, Countess of Lovelace, who first realized it was general-purpose and published the first algorithm for it,
thus making Babbage the first computer engineer and Lovelace the first computer scientist.
Babbage's machine was never completed and no subsequent work built on Babbage and Lovelace's pioneering efforts.

In 1933 Kurt Gödel and Jacques Herbrand defined a class of general recursive functions;
in 1936 Alonzo Church expanded on this to define the lambda calculus;
and later in 1936 Alan Turing defined an abstract notion of a universal computing machine, later known as the Turing Machine.
In his 1939 Ph.D. dissertation Turing (with his Ph.D. advisor Alonzo Church)
proposed that these three are all equivalent, and that no more computationally-powerful machine can exist.
Known as the Church-Turing thesis, the postulate that no more powerful machine can exist cannot be proven but is generally acknowledged as true.
Of the three models, Turing's had the advantage that Babbage's had had 100 years earlier
of being a description of a plausibly constructable machine,
but was not realistic enough to actually be constructed until decades later, when people started creating them as historical novelties. 

Although many people were involved in the development of functional universal computing machines^[There is indirect evidence that many of them were anonymous women: many women in part because most people with the profession "computer" were women and because there was a shortage of male labor during the second world war when computers were being pioneered; anonymous because the culture of the day did not generally acknowledge female input or provide scope for their leadership.],
the clearest progenitor to how most computers are designed
is John von Neumann, as described in the 1945 publication *First Draft of a Report on the EDVAC*.
This model is the subject of our next section.

# Von Neumann architecture

The von Neumann architecture consists of the following parts:

- The **memory** unit, which acts like an array or list of numbers.
    Given an **address** (an index into the array)
    it gives back the number stored there;
    given an address and a number, it stores the number at that address.

- The **control unit**, which has several parts:
    - A **program counter** (sometimes also known as an *instruction pointer*),
        a register that holds the address where the next instruction is to be found in memory;
    - an **instruction register** that holds the *instruction* currently being executed;
    - logic to adjust the program counter based on its current contents and the contents of the instruction register; and
    - logic to control what the ALU does and how to send data to and from memory based on the contents of the instruction register.

- The **arithmetic logic unit** (usually abbreviated as **ALU**), consisting of
    - A small number of **program registers** for storing operands and results; and
    - a set of logical gates implementing basic arithmetic operations using those registers as inputs and outputs.

- Ability to communicate with external devices, such as disks, keyboards, screens, etc.


{.exercise ...}
Write the basic von Neumann architecture simulator in the programming language of your choice.
You should include:

- A list or array of 256 `0`s as a representation of `memory`,
    in such a way you can easily change this to have different specific values inside it.

- A list or array of 4 `0`s as a representation of the program `registers`.

- Two variables, `pc` and `ir` to store the contents of the program counter and instruction register, respectively.

- A `newPC` function that returns the value the `pc` should have next based on the current `pc` and `ir`; you can just return `-1` for now.

- A `work` function that uses the contents of the `ir` to decide what to do at each step; you can have it do nothing at all for now.

- A `cycle` function that does the following:
    - simulate a rising clock edge by
        - loading register inputs into registers
        - loading `newPC()` into `pc`
    - load `memory[pc]` into `ir`
    - `work()`

- A `run` function that repeatedly:
    - prints the value of `pc`
    - calls `cycle`
    - prints the value of `ir` and `registers`
    - stops if `pc` is negative or larger than `memory` can access

If you run the `run` function you should see something like

````
pc = 0
ir = 0; r0=0, r1=0, r2=0, r3=0
````

We'll come back to this basic simulator and flesh it out in subsequent sections.
{/}


# Instruction-set architecture

The core idea of a general-purpose computer is that the contents of memory describe the actions to take.
But memory just stores bytes, and there is no intrinsic right set of things to do on these bytes.
Various computational theorists have proven than even very limited sets of operations can be "complete"
in the sense that they can be used to emulate everything else,
but the general model is to have a range of actions that let programmers easily make the computer do what they wish.

The set of instructions a computer provides, together with how those instructions are encoded,
defines the computer's **Instruction Set Architecture** or **ISA**.

## The Basics

The most common three categories of actions are 

- moves (corresponding to assignment operators in most programming languages),
- maths (corresponding to mathematical operators in most programming languages), and
- jumps (corresponding to control constructs in most programming languages).

### Moves

One of the pillars of the imperative programming paradigm^[It is likely that if you have learned to program but have not learned about programming paradigms, you learned a variant of the imperative paradigm. There are other approaches to programming that do not depend on variable assignment, the two most famous being the functional paradigm (where recursion is the key to expressivity) and the logical paradigm (where power is obtained from clever application of horn clause reduction rules in formal logic). Computer hardware that uses these paradigms directly is unknown to the author of this text.] is the assignment operator `=`.
All it does is take a value which exists in one place and put it into another place,
but as you have already seen that operation allows a great deal of flexibility.

When building a machine, making circuitry that is as general as programming's `=` is not trivially possible.
Recall that our `work()` function must decide what to do based only on information it can see,
generally meaning the number stored in `ir`.
`register[0] = register[1]` is going to have to have a different number for it than is `register[2] = register[7]`;
and both will be different from `register[3] = memory[131]` or `register[3] = memory[register[0]]`.

That said, once we enumerate all of the moves we think are important,
we can make them all happen in `work()` with a long but simple `if`-statement,
or equivalently with a large mux in hardware.

### Maths

After `=`, the next building block of most imperative programming languages is binary operators.
These are big networks of gates: `+` we've already looked at, and `-` is very similar;
`*` is basically one `+` for each bit in the input numbers;
and `\\` and `%`, while quite a bit more complicated still, can still be handled with a lot of gates.

There are other operators we'll need too, like `>=` and its friends,
and [boolean operators](#bit-wise-boolean-operators-in-code) as well,
but in the end they're all just a bunch of gates.

To make the machine use them, though, we have an additional complexity.
If there were a lot of ways to combine a source and destination for moves,
there are even more ways to combine two sources and a destination for maths.
Even using just 8 registers, there are 8^3^ = 512 possible combinations of `register[?] = register[?] + register[?]`,
and more when we get to operands in memory or literals.

That said, the `work()` for maths is not dramatically different from the `work()` for move;
there is a lot more circuitry in `if (ir == 12345): register[2] = register[3] * register[4]` than in `if (ir == 12345): register[2] = register[3]`,
but the logic needed to make it work is still basically a big mux.

### Jumps

While moves and maths have direct correlation to components of common programming languages,
jumps are somewhat less direct.

The concept of a jump is a program instruction that specifies the new `pc` as something other than the next instruction.
These can enable loops:

+-------------------------+-----------------------------------+
|     repeat forever:     |        1. do stuff                |
|         do stuff        |        2. more stuff to           |
|         more stuff too  |        3. run instruction 1 next  |
+-------------------------+-----------------------------------+

and conditionals

+-------------------------+-----------------------------------+
|     if x < y:           |        1. if not(x < y) skip to 3 |
|         do stuff        |        2. do stuff                |
|     more stuff too      |        3. more stuff too          |
+-------------------------+-----------------------------------+

and also, by combining those, all the other kind of control constructs:
`while` loops are loops with a conditional redirection,
function calls and returns are (mostly just) jumps to and from the function code,
etc.

There is no `work()` in most jumps;
instead, they add a mux to the `newPC()` function.

## Encoding instructions

There is no one "right" way to encode instructions into binary,
but it is common to do this by having particular sets of bits with particular meanings.
For example, we can specify one of 4 registers using 2 bits;
one of 8 operations using 3 bits; etc.
Thus, a simple way to create an instruction encoding is to figure out all the information any action will need, figure out how many bits we need to encode that, and then reserve a set of bits in the instruction for that information.

{.example ...}
Suppose we want the following operations:

1. Move a literal value into a register
1. Jump to a specific address if a given register is non-zero
1. Move into a register a value found at a memory address stored in another register
1. Move a value from a register into a memory address stored in another register
1. `a = a op b` where `op` is one of `+`, `-`, `*`, and `<`; and `a` and `b` are registers

That's eight operations, so we need 3 bits to encode which one is which.
We also need 4 bits to encode up 2 registers
and 8 more to encode a literal value, for a total of 15 bits.
15 bits fits into 2 bytes, so we can make our encoding be the following:

1. Look up the high-order 3 bits of the first byte on the following table:
    
    bits    meaning
    ------  --------------------------------------------------------------------
    `000`   move a literal into a register
    `001`   jump to address if register non-zero
    `010`   move from memory to register
    `011`   move from register to memory
    `100`   add one register to another
    `101`   subtract one register from another
    `110`   multiply one register by another
    `111`   set one register to 1 if it is less than another, 0 otherwise
    
2. The low-order 4 bits of the first byte give two registers;
    if the action changes a register, that is given by the higher-order two bits;
    if a register is not changed, it is given in the lower-order two bits.

3. The second byte is the literal value (for `000`) or address (for `001`).

This is of course not the only encoding we could make, but it does contain all the needed information.

Using this encoding, to encode

    x = 3
    while x < 50
        x *= x

we'd first translate that into steps our example ISA has access to

1. r0 = 3
2. r1 = 50
3. r0 = r0 × r0
4. r2 = r0
    
    since we don't have this operation, we'll do it as
    
    a. r2 = 0
    b. r2 = r2 + r0
5. r2 = 1 if r2 < r1 else 0
6. if (r2 ≠ 0) jump to step 3 

and then encode those instructions into binary (using `_` for unspecified bits

    000 00__ 00000011
    000 01__ 00110010
    110 0000
    000 1000 00000000
    100 1000
    111 1001
    001 10__ ????????

Finally we have to decide where in memory we are putting this so we know what value to put into the `????????` jump address. Let's put it at address 0, so the index of the multiply instruction is 4

    001___10 00000100

Now we assume all `_` are 0, prepend a 1 bit to each instruction, and turn that into hex so we can put it into a simulator and get the following contents of memory:

    0x80,0x03, 0x84,0x32, 0xE0, 0x88,0x00, 0xC8, 0xF9, 0x98,0x04
{/}

{.exercise ...}
Create an encoding for the following code that computes `r1 = r2 / r3` by implementing the following:

    r1 = 0
    while r1 * r2 < r3
        r1 += 1

We can change the loop to a jump by doing

1. `r1 = -1`
2. `r1 += 1`
3. if `r1 * r2 < r3`, go to step 2 again

and reduce that to the few operations we have available to us

1. r2 = *a numerator*
1. r3 = *a denominator*
1. r1 = −1
1. r0 = 1
1. r1 = r1 + r0
1. r0 = r1
1. r0 = r0 × r2
1. r0 = 1 if r0 < r3 else 0
1. if (r0 ≠ 0) jump to step 4

Encode the above sequence of instructions into bytes.
Remember to use the [two's complement encoding](bits.html#negative-numbers) of −1.
{/}

{.exercise ...}
Modify your simulator from the last exercise to implement the example instruction set encoding listed above.
In particular,

-   Refer to the section on [Masks](bool.html#masks) to remember how to get specific bits out of a number in code.
-   In `loop()` we'll still have `ir` become `memory[pc]` (the first byte only) for simplicity.
-   `newPC` will generally return `pc + 2` since instructions are 2 bytes long;
    but if the top three bits of `ir` are `000` and the register indicated by the low-order two bits of `ir` is not `0`, it will return the second byte of the instruction (i.e., `memory[pc+1]`) instead.
-   `work()` will need to consider the high-order three bits of `ir` and act as per the table in the example above.

Verify that your simulator runs the two example programs above.
The code from the example should put 81 (0x51) into `r0`;
the code from the exercise should put `r2 / r3` into `r1`.
{/}

### Fixed- or variable-length

Not all instructions need the same amount of information to be represented.
Should instructions that need less information take up fewer bytes than instructions that need a lot of information?

For such a simple question, there does not appear to be a simple answer.
Fixed-length instructions make some of what the computer does simpler, theoretically providing cost and power benefits;
variable-length instructions make more efficient use of memory, also theoretically providing cost and power benefits.
Intel and AMD are known for variable-length instruction sets,
while IBM and Motorola have more fixed-length instruction sets.

{.exercise ...}
Modify your simulator so it increases the `pc` by 2 only if the instruction needs 2 bytes (i.e., `000` and `001`); otherwise only increase the `pc` by 1.

This should let us shorten our example program

    0x00,0x03, 0x04,0x32, 0xC0,0x00, 0x68,0x00, 0xE9,0x00, 0x22,0x04

to just

    0x00,0x03, 0x04,0x32, 0xC0, 0x68, 0xE9, 0x22,0x04

removing 3 of the original 12 bytes (a 25% space saving).
How much space does this save in the division program?
{/}


## Addressing modes

How should we be able to represent operands of operations?
Many different representations are useful, leading to a a diversity of **addressing modes**:

Register
:   The most fundamental and universal addressing mode is to operate on values stored in registers.

Immediate
:   Immediate addressing refers to putting the value directly in the instruction.
    This roughly correlates to using literals in source code.
    
    Immediate addressing is never used for destinations of operations.
    Theoretically it would be *possible* to store a result into the program memory itself,
    but this is not done by any mainstream ISA.

Memory
:   Because access to memory is generally through a low-capacity interface,
    most ISAs limit at most one operand to be in memory
    and some limit all memory addressing to move operations only.
    
    There are multiple sub-types of memory addressing.
    The three basic building-blocks are:
    
    Immediate address
    :   The address is available as a literal value in the code.
        This is useful for encoding global variables and function addresses.
    
    Register address
    :   The address is available as the contents of a program register.
        This is useful for passing references to mutable values in variables.
    
    Relative address
    :   For various reasons (space efficiency, relocatable code, etc)
        it is sometimes convenient to compute addresses relative to some reference address,
        such as the current program counter or a dedicated base address register.

    These three blocks can be usefully combined in various ways.
    For example, one common addressing mode in Intel's x86 architecture
    is "Relative Immediate~1~ + Register~1~ + Register~2~ × 2^Immediate~2~^", where Immediate~2~ is limited to numbers no larger than 4.
    While that may seem strangely specific and ideosyncratic, 
    it is useful for several common code patterns, such as `x[i].y`.

Picking the right set of addressing modes in an ISA is challenging.
Fixed-length encodings in particular often face problems in these,
as something as simple as a plain immediate value requires that instructions contain more bits
than does any legal constant.
To void extra code bloat to make every instruction large enough to fit a full-length immediate value,
mainstream ISAs that use fixed-length encoding generally can represent only a small subset of integer literals as immediate values and require all others to be constructed by instruction sequences,
sometimes adding customized instructions to enable that construction to be relatively efficient.
Variable-length ISAs do not suffer from that limitation, but generally have quite complicated rules on how to determine the meaning of each instruction anyway so there is little gain in clarity.


## Extra State

In addition to program registers and memory, processors generally store additional information.
We've already seen a few of these: the PC and IR are registers
but not program registers because they cannot be directly addressed by programs.
Modern processors have hundreds of other similar hidden support state,
a few of which are important enough to deserve additional conversation.

### Condition codes

Many processors have a set of registers set automatically by some operations, called **condition codes**.
The can be used to determine how certain other operations behave.

In the x86 series of ISAs, for example, every mathematical operation
not only computes its resulting value
but also sets several condition code bits that describe the result of the computation.
These are used to determine if all conditional jumps (and other conditional operations) are executed or not.
Thus instead of writing

    r0 = 1 if r0 < r3 else 0
    if (r0 ≠ 0) jump to step 4

in x86 you'd write the equivalent of

    r0 -= r3
    if (last result < 0) jump to step 4

The ARM family of ISAs also use similar condition codes,
but unlike x86 where only a few instructions are conditional,
in ARM 7 almost every instruction is conditional.

Other ISAs, such as PowerPC and MIPS,
do not make as extensive use of condition codes,
instead using explicit condition operands in conditional operations.
Even these still have some condition codes, however,
to track things like integer overflow and other unusual conditions.

### The stack

There was a period in which enough code was written directly in machine language
that hardware developers chose to start adding higher-level abstractions
directly into their ISAs.
While all of these still exist in older ISAs like x86, most are no longer much used.
One, however, is very pervasive: the stack operations.

The concept of a stack, as implemented by the x86 ISA, is easier to explain in code than words:

    stack_memory = big contiguous chunk of memory
    stack_pointer = length_of(stack_memory)
    
    how to push(value):
      1. stack_pointer -= 1
      2. stack_memory[stack_pointer] = value
    
    how to pop():
      1. result = stack_memory[stack_pointer]
      2. stack_pointer += 1
      3. return result

{.example ...}
Consider the following code:

    push(3)
    push(4)
    push(5)
    push(6)
    pop()
    x = pop()
    pop()

At the end of the code `x` contains the value `5`
and the stack still contains one value (the `3`).
{/}

{.exercise ...}
Consider the following code:

    push(3)
    push(4)
    x = pop()
    push(5)
    y = pop()
    z = pop()

What are in `x`, `y`, and `z` at the end of the code?^[Answer: x = 4, y = 5, z = 3]
{/}

In x86-64, the `stack_pointer` in the above pseudocode is a program register, `rsp`.
The ISA instruction `pop rax` does the equivalent of `rax = pop()` in the above pseudocode: that is, it reads a value from memory, stores that value in `rax`, and also increases the value in `rsp`.
`push` and `pop`, as well as related instructions like `call` and `ret`, are extremely prevalent in code in the wild today.

### Complicated operands

For various reasons, some historical and some performance-based,
some instructions assume particular registers are part of their operation.
This is particularly common in x86 and x86-64.

{.example ...}
Intel's ISA manual, volume 2, defines the 64-bit version of the `div` instruction
as dividing `rdx:rax` by the source operand and storing the quotient in `rax`, the remainder in `rdx`.
In other words, this is a 128-bit integer divided by a 64-bit integer
with both result and remainder returned,
using three source registers and two destination registers;
but only one of the source registers is able to be specified.

This results in assembly code for `answer = numerator / denominator` looking something like

- move numerator into `rax`
- zero out `rdx`
- `idiv` denominator
- move `rax` into answer
{/}

It is also fairly common to have the computer processor logically or physically divided into distinct subprocessors, each with a specialized set of instructions and registers.
x86, for example, uses separate registers for floating-point maths than for integer maths, and has more registers for various vector operations and so on.

## RISC vs CSIC

In the 1990s it was common for computer architecture textbooks to describe two kinds of ISAs:
the RISC (reduced instruction set) and the CISC (complex instruction set).
These do not describe any major production ISAs, but are still common terms in computer architecture discussions.
A summary follows:

+---------------------------+-------------------------------+------------+
|RISC                       |CISC                           |Today we see|
+===========================+===============================+============+
|Fixed-length encoding      |Variable-length encoding       |Both        |
+---------------------------+-------------------------------+------------+
|Few, simple instructions   |Many, complicated instructions |Mostly CISC |
+---------------------------+-------------------------------+------------+
|Few addressing modes       |Many addressing modes          |More CISC   |
+---------------------------+-------------------------------+------------+
|Uniform operand notation   |Idiosyncratic operand notation |Both        |
+---------------------------+-------------------------------+------------+
|Many program registers     |Few program registers          |More RISC   |
+---------------------------+-------------------------------+------------+
|Clean new design           |Backwards-compatible design    |Both        |
+---------------------------+-------------------------------+------------+
|All program registers are  |Some operations only work with |More CISC   |
|interchangeable            |specific registers             |            |
+---------------------------+-------------------------------+------------+
|1 instruction = 1 cycle    |some instructions take several |CISC        |
|                           |cycles                         |            |
+---------------------------+-------------------------------+------------+
|Immediate values cannot    |Immediate values can represent |Both        |
|represent all numbers      |all numbers                    |            |
+---------------------------+-------------------------------+------------+
|All maths operands are     |One maths operand may be in    |Both        |
|registers                  |memory                         |            |
+---------------------------+-------------------------------+------------+
|Some instructions may not  |No limitations on instruction  |More CISC   |
|follow others              |ordering                       |            |
+---------------------------+-------------------------------+------------+
|Conditions are in registers|There are special "condition   |More CISC   |
|                           |codes                          |            |
+---------------------------+-------------------------------+------------+
|Function arguments usually |Function arguments usually     |More RISC   |
|passed in registers        |passed on the stack            |            |
+---------------------------+-------------------------------+------------+

There are also architectures with significant design decisions beyond those in this table, such as the compiler-controlled instruction scheduling of the Itanium architecture, but they are beyond the scope of this course.


# Pipelining and beyond

Executing most instructions involved several steps, some of which depend on others.
For example, the `push` instruction involves the following:

<svg viewBox="0 0 674 404" xmlns="http://www.w3.org/2000/svg" style="max-width:40em" >
<g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(-8 400)">
<text text-anchor="middle" x="453.5" y="-374.3" font-size="16">Read memory at PC into IR</text>
<text text-anchor="middle" x="453.5" y="-302.3" font-size="16">Identify the instruction</text>
<path fill="none" stroke="black" d="M453.5,-359.697C453.5,-351.983 453.5,-342.712 453.5,-334.112"/>
<polygon fill="black" stroke="black" points="457,-334.104 453.5,-324.104 450,-334.104 457,-334.104"/>
<text text-anchor="middle" x="352.5" y="-230.3" font-size="16">Parse out operands</text>
<path fill="none" stroke="black" d="M428.793,-287.876C415.831,-278.893 399.772,-267.763 385.735,-258.034"/>
<polygon fill="black" stroke="black" points="387.515,-255.009 377.302,-252.19 383.528,-260.763 387.515,-255.009"/>
<text text-anchor="middle" x="554.5" y="-230.3" font-size="16">Compute next PC</text>
<path fill="none" stroke="black" d="M478.207,-287.876C491.169,-278.893 507.228,-267.763 521.265,-258.034"/>
<polygon fill="black" stroke="black" points="523.472,-260.763 529.698,-252.19 519.485,-255.009 523.472,-260.763"/>
<text text-anchor="middle" x="256.5" y="-158.3" font-size="16">Read program registers</text>
<path fill="none" stroke="black" d="M328.77,-215.697C316.563,-206.796 301.515,-195.823 288.316,-186.199"/>
<polygon fill="black" stroke="black" points="290.1,-183.168 279.957,-180.104 285.975,-188.824 290.1,-183.168"/>
<text text-anchor="middle" x="400.5" y="-14.3" font-size="16">Write stack pointer to register</text>
<path fill="none" stroke="black" d="M358.539,-215.966C362.053,-205.648 366.388,-192.171 369.5,-180 381.409,-133.419 391.098,-78.2187 396.312,-46.0503"/>
<polygon fill="black" stroke="black" points="399.785,-46.4966 397.906,-36.0698 392.873,-45.3928 399.785,-46.4966"/>
<text text-anchor="middle" x="554.5" y="-158.3" font-size="16">Send next PC to PC register</text>
<path fill="none" stroke="black" d="M554.5,-215.697C554.5,-207.983 554.5,-198.712 554.5,-190.112"/>
<polygon fill="black" stroke="black" points="558,-190.104 554.5,-180.104 551,-190.104 558,-190.104"/>
<text text-anchor="middle" x="256.5" y="-86.3" font-size="16">Compute stack pointer</text>
<path fill="none" stroke="black" d="M256.5,-143.697C256.5,-135.983 256.5,-126.712 256.5,-118.112"/>
<polygon fill="black" stroke="black" points="260,-118.104 256.5,-108.104 253,-118.104 260,-118.104"/>
<text text-anchor="middle" x="139.5" y="-14.3" font-size="16">Write memory at stack address</text>
<path fill="none" stroke="black" d="M210.539,-143.787C193.563,-135.35 175.548,-123.585 163.5,-108 149.772,-90.2423 143.883,-65.2283 141.363,-46.239"/>
<polygon fill="black" stroke="black" points="144.824,-45.6856 140.258,-36.1248 137.865,-46.4457 144.824,-45.6856"/>
<path fill="none" stroke="black" d="M227.879,-71.8761C212.581,-62.7236 193.557,-51.3417 177.082,-41.485"/>
<polygon fill="black" stroke="black" points="178.61,-38.3205 168.231,-36.1898 175.016,-44.3275 178.61,-38.3205"/>
<path fill="none" stroke="black" d="M291.726,-71.8761C311.077,-62.4693 335.273,-50.7077 355.927,-40.6672"/>
<polygon fill="black" stroke="black" points="357.675,-43.7095 365.138,-36.1898 354.614,-37.4139 357.675,-43.7095"/>
</g>
</svg>

While each instruction has its own dependency graph of steps, they can all be fit into the following sequence:

1. Fetch an instruction from memory
2. Decode what the instruction intends to do
3. Retrieve values from the register file
4. Execute any math or logic operations the instruction requires
5. Access memory
6. Update values in the register file

{.aside ...} Naming the steps.

Every computer architecture text I have seen names the steps in the above sequence,
but the exact set of steps and names, as well as the set of work included in each step,
varies.
The names "Fetch", "Decode", and "Execute" are almost always included,
but which one includes accessing program registers varies.
Some texts also name steps "Memory", "Writeback", and/or "PC Update".

Official ISA documentation also tends to use these names in some form,
though often with additional steps as well.
{/}

This decomposition of work into steps is not merely academic;
processor throughput of instructions can be dramatically increased by **pipelining** these steps.
"Pipeline" is the name used in hardware design for the manufacturing notion of an assembly line:
when the part of the chip that fetches instructions finishes fetching the first instruction
it passes it off to the next part of the chip and proceeds to fetch the second instruction.
Thus, at any given time the processor might have many instructions in different stages of completion.

There are many additional issues that arise when pipelining.
A few of the more common techniques used to resolve these issues include:

Stall
:   Instruction dependencies may need to cause one stage to stand idle, waiting for an earlier instruction in a later stage to finish.

Forward
:   One instruction may grab information from a half-completed instruction further down the pipeline.

Reorder
:   A later instruction may be executed before an earlier one to give otherwise-stalled stages something to do.
