---
title: Boolean Algebra and Gates
author: Luther Tychonievich
license: <a href="https://creativecommons.org/licenses/by-nc-sa/4.0/">Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0)</a>
...

Because all information can be reduced to a sequence of bits,
it is useful to consider what operations can be performed on bits.

When discussing individual bits, several different terminologies are used.
Sometimes we refer to `1` as a "set" bit and `0` as a "cleared" bit;
sometimes `1` is called "true" and `0` "false";
and sometimes `1` is called "high" and `0` "low".
Each set of terms makes more sense than the others in some situations,
which is why all three sets continue to be used today.
We'll use all three interchangeably in this chapter.

The basic operations on bits were discussed extensively by George Boole
and the rules governing them are eponymously named "Boolean algebra."
This algebra is widely used in the branch of philosophy called "formal logic."
It is also the foundation of digital circuit design,
where it is represented in terms of wires, voltage on those wires, and gates.
And there is a common syntax for Boolean algebra shared by many programming languages^[One notable exception is Python, which has the same bitwise syntax but uses `not`{.python} instead of `!`, `and`{.python} instead of `&&`, `or`{.python} instead of `||`, and `p if a else q`{.python} instead of `a ? p : q`.],
which has two versions of each operation with slightly different meaning.
This chapter makes an effort to present all^[There are multiple styles of circuit diagrams; this chapter uses the ANSI gates, not DIN or IEC.] of these perspectives together.

# Core operations

There is only one meaningful operation that takes as input a single bit.

Not
:   The "not" operation takes one input bit and produces one output bit.
    It is represented using the following symbols

    --------------- -----------
    Formal logic    ¬P *or* <span style="text-decoration:overline">P</span>
    Code, bitwise   `~p`
    Code, logical   `!p`
    Circuits[^not]  ![](img/not.svg){height=2em}
    --------------- -----------

    The output is the opposite of the input value; this is enumerated in the following table:

     `p`   `!p`
    ----- ------
      0    1
      1    0

    The "not" operator is sometimes said to "negate", "invert", or "flip" its input.

[^not]: Technically, the small circle is the "not" part; the triangle is sort of like a filler for when there is no other gate to which the circle may be adjoined. 

{.aside ...} The code symbol for "bitwise not" is `~`, the "tilde", which looks a lot like the "similar to" symbol ∼, the "most positive" symbol ∾, and the "alternating current" symbol ∿; fortunately, which one is meant can usually be inferred from context. The code symbol also has inconsistent representation across typefaces, sometimes being almost indistinguishable from a dash `−` or rendered as a small diacritic mark `˜`.
{/}

There are two commonly identified operations that take as input two bits.

And
:   The "and" operation takes two input bits and produces one output bit.
    It is represented using the following symbols

    ---------------         -----------
    Formal logic            P∧Q
    Code, bitwise           `p & q`
    Code, logical[^short]   `p && q`
    Circuits                ![](img/and.svg){height=2em}
    ---------------         -----------

    The output is `1` when both inputs are `1`; this is enumerated in the following table:

     `p`   `q`   `p & q`
    ----- ----- ---------
     0     0       0
     0     1       0
     1     0       0
     1     1       1

    The "and" operator is sometimes said to find the "conjunction" of its inputs.

Or
:   The "or" operation takes two input bits and produces one output bit.
    It is represented using the following symbols

    ---------------         -----------
    Formal logic            P∨Q
    Code, bitwise           `p | q`
    Code, logical[^short]   `p || q`
    Circuits                ![](img/or.svg){height=2em}
    ---------------         -----------

    The output is `1` when either input is `1`; this is enumerated in the following table:

     `p`   `q`   `p | q`
    ----- ----- ---------
     0     0       0
     0     1       1
     1     0       1
     1     1       1

    The "or" operator is sometimes said to find the "disjunction" of its inputs.
    It is also sometimes called the "inclusive or" (<var>p</var> or <var>q</var> or both) to distinguish it from the "exclusive or"  (<var>p</var> or <var>q</var> but not both).

[^short]: `&&` and `||` are typically implemented in code as "short-circuit" operators.
    This means that `f() && g()` will only execute `g()` if `f()` was true
    and only execute `f() || g()` if `f()` was false.
    
    Dynamically-typed programming languages, where an expression is allowed to return different type each time it is executed,
    often add to `&&` and `||` additional behavior:
    `&&` will return its first operand if it is false, otherwise its second;
    and `||` will return its first operand if it is true, otherwise its second.
    Thus, for example, `0 && true` returns `0`,
    `23 || null` returns `23`,
    `0.0 || "stuff"` returns `"stuff"`,
    etc.

One of the principle theorems of Boolean algebra is that all other operations on bits, of any number of inputs, can be expressed in terms of the above three basic operations.



# Additional named operations

There are several additional Boolean operations that are sometimes encountered:


Nand and Nor
:   The "Nand" and "nor" operations are equivalent to the "and" and "or" operations followed by a "not" operation.
    They are primarily used in digital circuits, where they can generally be implemented
    with fewer transistors than the combined circuit could be.

    --------------- -----------
    Circuits (nor)  ![](img/nor.svg){height=2em}
    Circuits (nand) ![](img/nand.svg){height=2em}
    --------------- -----------

    The meaning of "P nand Q" is the same as `~(p&q)`;
    the meaning of "P nor Q" the same as "`~(p|q)`".

Exclusive Or
:   The "exclusive or" or "xor" operation takes two input bits and produces one output bit.
    It is represented using the following symbols

    --------------- -----------
    Formal logic    P⊕Q
    Code, bitwise   `p ^ q`
    Circuits        ![](img/xor.svg){height=2em}
    --------------- -----------

    The output is `1` when exactly one input is `1`; this is enumerated in the following table:

     `p`   `q`   `p ^ q`
    ----- ----- ---------
     0     0       0
     0     1       1
     1     0       1
     1     1       0

     "Exclusive or" is can also be created from the basic three gates: `p ^ q` is equivalent to `(p | q) & ~(p & q)` or `(p & ~q) | (q & ~p)`.

     Exclusive or is also the operation that computes parity, as discussed [above](#parity-checksums-error-correction-codes-and-digests).

{.aside ...} The code symbol for "exclusive or" is `^`, the "carat" or "circumflex", which looks a lot like the logical symbol for "and": ∧, the "wedge"; unfortunately, context does not help much in knowing which was meant. The code symbol also has inconsistent representation across typefaces, varying in size and position; some represent it almost like a capital lambda `Λ`, while others render it as a small diacritic mark `ˆ`.
{/}

Implies

:   Although it is more common in formal logic than circuits and computing,
    implication can also be seen as a Boolean gate.
    If "P implies Q" to be true, then it is not possible for P to be true and Q false.
    In other words, either P is false or Q is true: `~p | q`.

    ------------- -----------
    Formal logic  P ⇒ Q *or* P ⊢ Q
    Circuits      ![](img/imply.svg){height=2em}
    ------------- -----------


Multiplex
:   An example of a common three-input Boolean gate is the "multiplexer" or "mux".
    A mux uses one of its inputs to decide which of the other two to output:

     `s`   `s ? p : q`
    ----- -------------
     0     `q`
     1     `p`

    That can also be written out in full:
    
     `s`   `p`   `q`   `s ? p : q`
    ----- ----- ----- -------------
      0     0     0         0
      0     0     1         1
      0     1     0         0
      0     1     1         1
      1     0     0         0
      1     0     1         0
      1     1     0         1
      1     1     1         1


    A bit-wise mux (which is not the most common type) can be written as `(s&p) | ((~s)&q)`;
    the usual logical meaning of `s ? p : q` is a bit more complicated because of some nuances of the logical operators in most programming languages[^trinary].

[^trinary]: Two considerations of `?:` are worth noting if you wish to use it in coding.
    
    There is a syntactic ambiguity in the code `a?b:c?d:e`; it could mean either `(a?b:c)?d:e` or `a?b:(c?d:e)`.
    Programming languages traditionally bind it in the second way.
    
    `b ? f() : g()` is traditionally implemented as a short-circuit operator,
    meaning that if `b` is false `f()` will be invoked by `g()` will not.
    

# Fancier logic

We can use the basic building blocks of logical gates to implement much more complex operations.
How that is done is more properly the domain of computer architecture,
but one example can help reveal that complicated logic can be implemented.

{.example ...} Suppose we have two binary numbers we wish to add, using only basic logical operations.
Each number is represented by a sequence of bits;
*x*~0~ is the 1s place of number *x*,
*x*~1~ is the 2s place,
*x*~2~ is the 4s place,
*x*~3~ is the 8s place, and so on;
similarly with *y*.
We want to arrange a set of individual Boolean operations to compute all of the bits of *z*, where *z* = *x* + *y*.

We'll proceed the same way we would by hand: with the least-significant digit first.
To be sure we catch all cases, let's enumerate all four possible combinations of *x*~0~ and *y*~0~ and what the *z*~0~ and carry should be in each case.

 *x*~0~   *y*~0~   *z*~0~   carry~1~
-------- -------- -------- ----------
   0        0         0       0
   0        1         1       0
   1        0         1       0
   1        1         0       1

Notice that the *z*~0~ column looks just like the "xor" table;
and that the carry~1~ column looks just like the "and" table.
Thus we can configure the following:

    z0 = x0 ^ y0
    c1 = x0 & y0

Now for *z*~1~.
This is the sum of *x*~1~, *y*~1~, and the carry we just computed.
Again for completeness, let's enumerate all 8 combinations possible for these three inputs:

 *c*~1~   *x*~1~   *y*~1~   *z*~1~   carry~2~
-------- -------- -------- -------- ----------
    0      0        0         0       0
    0      0        1         1       0
    0      1        0         1       0
    0      1        1         0       1
    1      0        0         1       0
    1      0        1         0       1
    1      1        0         0       1
    1      1        1         1       1

The *z*~1~ column is the parity of *c*~1~, *x*~1~, and *y*~1~, which can be computed by a pair of "xor"s:

    z1 = c1 ^ x1 ^ y1

The carry~2~ is more complicated, but notice that the entries when *c*~1~ is `0` are the "and" table and the entries when *c*~1~ is `1` are the "or" table. Thus we can use *c*~1~ like the selector of a mux:

    c2 = c1 ? (x1 | y1) : (x1 & y1)

There are other combinations that also work; for example

    c2 = (x1 & y1) | (c1 & (x1 ^ y1))

Everything we did for *z*~1~ and *c*~2~ also apply for all later *z*s and *c*s:

    z2 = c2 ^ x2 ^ y2
    c3 = (x2 & y2) | (c2 & (x2 ^ y2))
    z3 = c3 ^ x3 ^ y3
    c4 = (x3 & y3) | (c3 & (x3 ^ y3))
    z4 = c4 ^ x4 ^ y4
    c5 = (x4 & y4) | (c4 & (x4 ^ y4))
    ...

Thus, we can wire together a bunch of "and", "or", and "xor" gates to create an "adder."

![](img/adder.svg)
{/}

In general, any deterministic function with a fixed number of fixed-length binary inputs and a fixed number of fixed-length binary outputs can be implemented using some combination of Boolean logic gates.

# Bit-wise Boolean operators in code

Virtually every C-derived language, including Java, Python, Javascript, and most other languages in common use today,
have a set of bit-wise Boolean operators that can be combined to perform various tasks.
These treat the integer datatype in the language in question as an array or list of bits (generally 32 bits, though that varies a little) and allow you to manipulate them directly.

+---------+---------------------------+---------------------------------------------------------+
|Operator | Meaning                   | Example                                                 |
+:=======:+:==========================+:========================================================+
|`&`      | Bit-wise and              |  1100~2~ `&` 0110~2~ → 0100~2~<br/>(i.e., `12 & 6 == 4`)|
+---------+---------------------------+---------------------------------------------------------+
|`|`      | Bit-wise or               | 1100~2~ `|` 0110~2~ → 1110~2~<br/>(i.e., `12 | 6 == 14`)|
+---------+---------------------------+---------------------------------------------------------+
|`^`      | Bit-wise xor              | 1100~2~ `^` 0110~2~ → 1010~2~<br/>(i.e., `12 ^ 6 == 10`)|
+---------+---------------------------+---------------------------------------------------------+
|`>>`     | Bit-shift to the right    | 1101001~2~ `>>` 3 → 1101~2~<br/>(i.e., `105 >> 3 == 13`)|
+---------+---------------------------+---------------------------------------------------------+
|`<<`     | Bit-shift to the left     | 1101~2~ `<<` 3 → 1101000~2~<br/>(i.e., `13 << 3 == 104`)|
+---------+---------------------------+---------------------------------------------------------+

When shifting, bits that no longer fit within the number are dropped.
New bits are generally added to keep the number the same number of bits;
for left shifts those new bits are always 0s, but for right shifts they are sometimes 0s
and sometimes copies of whatever bit had been in the highest-order spot before the shift.
Copying the high-order bit is called "sign-extending" because it keeps negative numbers negative in twos-complement.
Which kind of right-shift is performed varies by language and by datatype shifted.
Some languages also have a third shift `>>>` to distinguish between sign-extending (`>>`) and zero-extending (`>>>`) right shifts.

## Masks

A bit-mask or simply **mask** is a value used to select a set of bits from another value.
Typically, these have a sequential set of bits set to 1 while all others are 0,
and are used with an `&` to select particular bits out of a value.

Bit-mask constants are generally written in hexadecimal; for example, `0x3ffe0` (or 0011 1111 1111 1110 0000~2~) selects 13 bits, the 5th-least-significant through the 27th.

Bit-mask computed values are generally built using shifts and negations;
for example, `((~0)<<5) ^ ((~0)<<14)` generates `0x3fe0`:

Expression | binary | description | alternative constructions
-----------|-------:|-------------|---------------------------
`0` | ...`00000000000000000` | all zeros |
`~0` | ...`11111111111111111` | all ones | `-1`
`(~0)<<5` | ...`11111111111100000` | ones with 5 zeros in the bottom place | `~((1<<5)-1)`
`(~0)<<14` | ...`11100000000000000` | ones with 14 zeros in the bottom place | `~((1<<14)-1)`
`((~0)<<5) ^ ((~0)<<14)` | ...`00011111111100000` | 9 ones, 5 places from bottom | `((1<<9)-1)<<5`, `(~((~0)<<9))<<5`

## Bit terminology

When discussing a sequence of bits, some terms are used in multiple ways:

Bit Vector
:   A common name for a fixed-length sequence of bits,
    implemented using one of a programming language's built-in integer types,
    manipulated primarily by bit-wise operations.

    Also a name for a more complicated data structure that stores any number of one-bit values.

Clear
:   As a verb, either replace a single bit with 0 or replace all bits with 0.
    To clear the 4th bit of `x`, you'd do `x &= ~(1<<4)`.
    To clear `x`, you'd do `x &= 0`.
    
    As a noun, "is zero", usually of a specific bit.
    To check if the 4th bit of `x` is clear you'd do `(x & (1<<4)) == 0`.


*i*th bit
:   Usually the bit which, in a numeric interpretation, would be in the 2^*i*^s place
    (i.e., the 3rd bit is in the 8s place): in other words, counting from least- to most-significant
    starting at 0.
    A number with just the `k`th bit a 1 can be created as `1<<k`.
    Unless otherwise specified, this is the usage of bit ordinals throughout this text.
    
    Sometimes starts counting from the most- instead of least-significant bit.
    
    Sometimes counts from 1 instead of 0.
    
    Sometimes people use "th" for all 0-based bit counting (e.g., "the 1th bit" instead of "the 1st bit")

Set
:   Sometimes a verb, "set this bit", meaning make it a 1.
    To set the 4th bit of `x`, you'd do `x |= 1<<4`.
    
    Sometimes an adjective, meaning a bit position containing a 1.
    Thus in the number 1100110~2~ the 2nd bit is set but the 3rd is not.

Zero
:   Sometimes the opposite of 1 as related to a single bit.

    Sometimes a bit vector of all 0 bits.
    
    Sometimes a synonym for "clear".
    The verb form is also sometimes rendered "zero out".


## Bit-sets and flags

One common practical use of bit manipulation in programming is the concise representation of sets of Boolean values.
Given a small fixed set of possible elements of a set, any particular set of those elements
can be efficiently represented by a number,
where each possible element is assigned a unique bit within a number.
For example, if our possible set of elements is {fun, important, required, useful, good prof, good time} we might say

````
FUN       = 1<<0
IMPORTANT = 1<<1
REQUIRED  = 1<<2
USEFUL    = 1<<3
GOOD_PROF = 1<<4
GOOD_TIME = 1<<5
````

and then `GOOD_TIME | FUN | GOOD_PROF` (i.e., 110001~2~ or 0x31 or 49) would represent a filler elective,
while 0xE (or 14 or `IMPORTANT | REQUIRED | USEFUL`) would represent a class you know will be good for you, even if you don't enjoy it.

Each of one-nonzero-bit value is called a **flag**^[There is a related concept used in circuit design called "one-hot encoding" which also has values with just one non-0 bit, but that term is rarely used in software.] and a set represented by the bitwise-or of zero or more flags is often just called "the flags."

Given a set represented as flags-variable `x`,

| Set operation     | Bit-wise parallel |
|-------------------|-------------------|
| *a* ∈ *x*         | `(A & x) != 0`    |
| {*a*} ∪ *x*       | `A | x`           |
| *x* ∖ {*a*}        | `x & ~A`          |

| Set datatype action | Bit-wise parallel |
|---------------------|-------------------|
| `x.contains(A)`     | `(x & A) != 0`    |
| `x.add(A)`          | `x |= A`          |
| `x.remove(A)`       | `x &= ~A`         |


## Bit-fiddling

"Bit-fiddling" is a colloquialism for using sequences of operations to achieve various bit-level transformations of values.
While rarely of intrinsic importance in programming,
they show up often enough in practice that they sometimes make it into technical interviews and the like.

{.example ...}
Suppose you want to retrieve *k* bits from *x*, starting at bit *i*.
You'd first shift *x* to the right so the last *i* bits are not there:

    x >>= i

and then mask out the last *k* bits

    x &= (1<<k)-1
{/}

{.example ...}
Suppose you wanted to compute the parity of a 32-bit vector *x*,
as described in the section [Parity, checksums, error-correction codes, and digests](bits.html#parity-checksums-error-correction-codes-and-digests).
You could brute-force it:

    parity = 0
    repeat 32 times:
        parity ^= (x&1)
        x >>= 1

That has a total of 32 xors, 32 ands, and 32 shifts.
We can do it much more efficiently than that.

Observe that xor is both transitive and associative; thus we can re-write
$$x_0 ⊕ x_1 ⊕ x_2 ⊕ x_3 ⊕ x_4 ⊕ x_5 ⊕ x_6 ⊕ x_7$$
using transitivity as 
$$x_0 ⊕ x_4 ⊕ x_1 ⊕ x_5 ⊕ x_2 ⊕ x_6 ⊕ x_3 ⊕ x_7$$
and using associativity as
$$(x_0 ⊕ x_4) ⊕ (x_1 ⊕ x_5) ⊕ (x_2 ⊕ x_6) ⊕ (x_3 ⊕ x_7)$$
and then compute the contents of all the parentheses at once via `x ^ (x>>4)`.
Repeating this kind of computation at scale, we have

    x ^= (x>>16)
    x ^= (x>>8)
    x ^= (x>>4)
    x ^= (x>>2)
    x ^= (x>>1)
    parity = (x & 1)

That's just 5 xors, 1 and, and 5 shifts.

A lot of bit-fiddling is about finding these kinds of shortcuts and tricks.
Software engineers often find these shortcuts distasteful and confusing,
but in some rare circumstanced bit fiddling can provide significant memory or speed benefits.
{/}

{.exercise ...}
Consider the following bit-fiddling code:

    x ^= y
    y ^= x
    x ^= y

1. Try this out with various initial values of `x` and `y` and write a description of what this code is doing.

2. Work out by hand the contents of each variable in terms of the original values of `x` and `y`;
    for example, after running the first line `x` contains $x_0 ⊕ y_0$.
    Using the identities $a ⊕ a = 0$ and $0 ⊕ x = x$, prove that your description from part 1 is true.
{/}
