---
title: Bits and Beyond
author: Luther Tychonievich
license: <a href="https://creativecommons.org/licenses/by-nc-sa/4.0/">Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0)</a>
...

Claude Shannon founded the field of information theory.
A core fact in information theory is that there is a basic unit of information,
called a "bit^[a portmanteau of "binary digit"]" or a "bit of entropy."
Roughly speaking, a "bit" is an amount of information that is about as surprising as the result of a single coin flip.
In the sentence "please pass the salt" the word "salt" has less than a bit of entropy; most of the time someone says "please pass the" the next word they say is "salt," so adding that word provides very little more information.
On the other hand, the word that comes after "is" in "Hello, my name is" has many bits of entropy; no matter what the word is, it was quite surprising.

{.exercise ...}
Claude Shannon performed an experiment to determine the bits of entropy
of the average letter in English.
We have you perform a similar experiment as part of [Lab 2](lab02-information-theory.html).
<!--
Measure the bits of entropy of a human language of your choice.

1. Find a large corpus of example text (the larger the better)
2. Write a program that repeatedly
    1. selects a random substring from the corpus, long enough to give a little context (perhaps 50 characters)
    2. prompt the user to type what the next character will be
    3. track the number of correct and incorrect guesses

After guessing correctly on *r* out of *g* total tries,
an estimate of the bits of entropy per character in the corpus is
log~2~(*g* ÷ *r*).

Not all languages have the same information per character.
Shannon suggested English had roughly 1 bit per letter^[Shannon, Cluade E. (1950), "Prediction and Entropy of Printed English", *Bell Systems Technical Journal* (3) pp. 50--64.],
while languages that use ideograms have many more.

It is likely that bits per second of speaking is more constant across languages,
which could probably be tested by measuring the bits per character of subtitles
and combing it with the timing information the subtitles contain,
but I am unaware of any published effort to do this or any related measurement.
-->
{/}

# Digital Information

How much information can we transmit over a wire?
If we put voltage on one end, because wire conducts well we'll very soon see the same voltage at the other end.
Presumably, the more precise our measurement of that voltage can be, the more information we can collect.
If we can distinguish between 1000 voltage levels, for example, we'll get log~2~(1000) = 10 bits of information per measurement; whereas a less sensitive voltmeter that can only distinguish between two voltage levels gets only log~2~(2) = 1 bit per reading.
This seems to suggest that the more precise I can make the voltage generator and the more sensitive the voltmeter, the more information I can transmit per reading.

The problem with this assumption is that it takes longer to transmit higher-resolution data.
This is partly a consequence of fundamental mathematical and physical laws, like the Heisenberg uncertainty principle,
but it is also more tellingly a consequence of living in a noisy world.
To tell the difference between 8.35 and 8.34 volts, you need to ensure that the impact of wire quality and the environment through which it passes contributes significantly less than 0.01 voltage error to the measurement; generally this requires watching the line of a while to see what is the noisy variation and what is the true signal.
By contrast, telling the difference between 10 volts and 0 volts is much simpler and much more robust against noise.
It is quite possible to make several dozen good 1-bit measurements in the time it'd take to make one 10-bit measurement.

This observation led to advances in digital signals:
signals composed of a large number of simple "digits^["Digit" comes from a Latin word meaning "finger" and, prior to the computer age, was used to refer to the ten basic numerals 0 through 9. "Digital" meaning "a signal categorized into one of a small number of distinct states" became common in the 1960s, though it was used by computing pioneers as early as the late 1930s.]" instead of one fine-grained "analog^["Analog" (or "Analogue" outside the USA) comes from a word meaning "similar to" or "along side", suggesting that an analog signal has some direct correlation to the thing it represents.]" signal.
We can communicate much more information with much less error
by transmitting a large number of single-bit signals
than we can by transmitting a smaller number of signals with more information in each.

# Saying Anything with Bits

Information theory asserts that any information can be presented in any a format that contains enough bits.
There are many interesting theorems and algorithms in that field about how this can best be done,
but for most of computing we only need to consider a handful of simple encodings, described in the subsections below.

## Place-value numbers

### Base-10, "decimal"

When you were still a small child you were taught a place-value based numbering system.
Likely at that age you never considered why we called it "place-value,"
but the name is intended to suggest that the value of each digit depends not only on the digit itself, but also on its placement within the string of digits.
Thus in  the number string `314109`, the first `1`'s value is a hundred times larger than the second `1`'s value.
If we write out the full place values

 10^5^   10^4^   10^3^   10^2^   10^1^   10^0^
------- ------- ------- ------- ------- -------
   3       1       4       1       0       9

we can see that the number's total value is 3 × 10^5^ + 10^4^ + 4 × 10^3^ + 10^2^ + 9 × 10^0^ or three-hundred fourteen thousand one hundred nine.

### Base-2, "binary"

There is no particular magic to the 10s in the above example.
Because it is easier to distinguish two things than ten, we might reasonably try to use 2s instead.
Thus, instead of the digits 0 through 10 − 1 = 9 we use the digits 0 through 2 − 1 = 1.
Thus in the base-2 number string `110101` the first `1`'s value is thirty-two times larger than the last `1`'s value.
If we write out the full place values

 2^5^    2^4^    2^3^    2^2^    2^1^    2^0^
------- ------- ------- ------- ------- -------
  1       1       0       1       0       1

we can see that the number's total value is  2^5^ + 2^4^ + 2^2^ + 2^0^ or fifty-three.

Base-2 numbers of this sort are widely used to convert sequences of single-bit data into a larger many-bit datum.
Indeed, we call a binary digit a "bit," the same word information theory uses to describe the fundamental unit of information,
and often refer to any single-bit signal as being a `1` or `0` even if it is more accurately a high or low voltage, the presence or absence of light in a fiber, a smooth or pitted region on an optical disc, or any of the wide range of other physical signals.

Math works the same in base-2 as it does in base-10:

                1      11      11    1 11    1 11
      1011    1011    1011    1011    1011    1011
    + 1001  + 1001  + 1001  + 1001  + 1001  + 1001
    ------  ------  ------  ------  ------  ------
                 0      00     100    0100   10100

Binary is useful for hardware because it takes less space and power
to distinguish between two voltage states (high and low)
than between three or more.
However, for most uses we find it more useful to cluster groups of bits together,
typically either in 4-bit clusters called "nibbles" or "nybbles"
or in 8-bit clusters called "bytes" or "octets."

### Hexadecimal

Base-16, or hexadecimal, is useful because humans can read it without getting as easily lost in the long sequence of `0`s and `1`s, but it has a trivial mapping to the binary that hardware actually stores.

Hexadecimal digits are taken from the set of nibbles: {`0`, `1`, `2`, `3`, `4`, `5`, `6`, `7`, `8`, `9`, `a`, `b`, `c`, `d`, `e`, `f`}.
Place-value notation is used with base 16.
Thus the number `d02`~16~ (also written `0xd02`{.c}) represents the number

  16^2^   16^1^  16^0^
------- ------- -------
  d       0       2

`d` × 16^2^ + `2` × 16^0^
= 13×256 + 2×1
= 3328 + 2
= 3330

Hexadecimal is particularly noteworthy because it is easy to convert to and from binary:
each nibble is simply 4 bits.
Thus

|   d |    0 |    2 |
|:---:|:----:|:----:|
|1101 | 0000 | 0010 |

Hexidecimal is commonly used by humans who want to communicate in binary without getting lost in the middle of a long string of 0s and 1s.

Math works the same in base-16 as it does in base-10:

                1       1       1    1  1    1  1
      d0b2    d0b2    d0b2    d0b2    d0b2    d0b2
    + 300f  + 300f  + 300f  + 300f  + 300f  + 300f
    ------  ------  ------  ------  ------  ------
                 1      c1     0c1    00c1   100c1


### Bytes or octets

Base-256 uses octets or bytes, each being 2 nibbles or 8 bits.
Octets are typically written using pairs of nibbles directly;
thus the digits are taken from the set of bytes: {`00`, `01`, `02`, ... `fd`, `fe`, `ff`}.
Humans almost never do math directly in base-256.

Octets are noteworthy because, while processor mathematical circuits use base-2,
most of the rest of computing is based on octets instead.
Memory, disk, network, datatypes, assembly---all used bytes, not bits, as the smallest unit^[This is a gross over-simplification. A serial connection communicates in bits, a parallel in a fixed number of bits equal to the number of wires, etc. However, most protocols and interfaces present a byte-based interface.] of communication.


## Base-2 logs and exponents

Power-of-two numbers show up a lot in hardware and hardware-interfacing software.
It is worth learning the vocabulary used to express them.

Value | base-10 | Short form | Pronounced
|:--|--:|:-:|--:|
2^10^ | 1024 | Ki | Kilo
2^20^ | 1,048,576 | Mi | Mega
2^30^ | 1,073,741,824 | Gi | Giga
2^40^ | 1,099,511,627,776 | Ti | Tera
2^50^ | 1,125,899,906,842,624 | Pi | Peta
2^60^ | 1,152,921,504,606,846,976 | Ei | Exa

In all cases above, the i is usually dropped to just say (e.g.) G instead of Gi.  The i clarifies that we mean the base-2 power, not the base-10 power.  G could mean either 2^30^ or 10^9^, numbers the differ by about 7%; which one is meant can only be guessed from context unless the clarifying i is present (which it usually is not).

Only the first three rows in the table above (K, M, and G) will show up often in this course.
The only entry in the base-10 column we expect you to learn is the first, 2^10^ = 1024.

2^27^ = 2^7^ 2^20^ = 128M.  This pattern works for any power of 2: the 1's digit of the exponent becomes a number, the 10's digit becomes a letter.  Thus

Value | Split | Written
|:--|:-:|--:|
2^27^ | 2^7^ 2^20^ | 128M
2^3^  | 2^3^ 2^0^  | 8
2^39^ | 2^9^ 2^30^ | 512G

If you memorize the values of 2^0^ through 2^9^, you can then do these exponentiations in your head.
That is worth memorizing; these things show up too often to be worth wasting thought on all the time; memorize them now and you'll have less cognitive load in the future.

Logarithms with base 2 (written log~2~(*n*) or lg(*n*) or sometimes just log(*n*)) do the inverse of this:
lg(64G) = lg(2^6^ 2^30^) = lg(2^36^) = 36.
Again, these show up so often that you should be able to do them automatically.


{.exercise ...}
Fill in the rest of the following table.

Exponent | Written As
|:--|:--|
17 | 128K
3 | <input type="text" style="width:8ex"/>
38 | <input type="text" style="width:8ex"/>
11 | <input type="text" style="width:8ex"/>
<input type="text" style="width:8ex"/> | 256M
<input type="text" style="width:8ex"/> | 16G
<input type="text" style="width:8ex"/> | 32

Answers are in this footnote^[8, 256G, 2K, 28, 34, 5]
{/}


## Negative numbers

We are accustomed to having a special symbol that tells us if a number is negative.
When communicating in bits with just two symbols, that is not an option,
so several other approaches to representing negative numbers are used.
Three such approaches are often taught in courses like this,
and three are actually used, but they are not the same three.
We'll explore the three that are common in hardware
each in terms of decimal numbers first, then see how they map to binary.

All of these solutions depend on access to a finite number of bits/digits/nibbles/bytes to store numbers in.
Our examples will assume we have four digits.

Two's Complement
:   This version picks a number (typically half of the maximum number we can write, rounded up)
    and decides that that number and all numbers bigger than it are negative.
    How negative? To understand that, we need to observe a ring-like behavior of numbers.

    What is `0000 - 0001` ? The answer is based on place-value subtraction's notion of borrowing:
    `0 - 1` is `9` borrow `1`.  We end up borrowing from all spots to get `0000 - 0001 = 9999`
    with a stray borrow "overflowing" and getting lost due to finite precision.
    Since `0000 - 0001` is obviously negative 1, we set `9999` to be negative 1.
    One less than negative 1 is negative 2 = `9998`, and so on.

    Two's complement is nice because the three most common mathematical operators (addition, subtraction, and multiplication)
    work the same for signed and unsigned values.
    Division is messier, but division is always messy.

    In hardware, two's complement is typically used for signed integers.

    <table><tbody><tr><td>
    <svg xmlns="http://www.w3.org/2000/svg" version="1.1" viewBox="0 0 150 150" style="width:18.5em">
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="15.324745" x="75.000122">
    <tspan y="15.324745" x="75.000122">0000</tspan>
    <tspan y="30.99356" x="75.00008">0</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="20.047983" x="97.967705">
    <tspan y="20.047983" x="97.967705">0001</tspan>
    <tspan y="34.53599" x="90.31180">+1</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="33.567291" x="117.4518">
    <tspan y="33.567291" x="117.4518">0010</tspan>
    <tspan y="44.67547" x="103.30120">+2</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="53.824379" x="130.48605">
    <tspan y="53.824379" x="130.48605">0011</tspan>
    <tspan y="59.86829" x="111.99070">+3</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="77.824837" x="135.00021">
    <tspan y="77.824837" x="135.00021">0100</tspan>
    <tspan y="77.86863" x="115.00014">+4</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="101.70831" x="130.43951">
    <tspan y="101.70831" x="130.43951">0101</tspan>
    <tspan y="95.78123" x="111.95967">+5</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="121.95574" x="117.4518">
    <tspan y="121.95574" x="117.4518">0110</tspan>
    <tspan y="110.96680" x="103.30120">+6</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="135.48457" x="98.014236">
    <tspan y="135.48457" x="98.014236">0111</tspan>
    <tspan y="121.11342" x="90.34282">+7</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="140.32494" x="75.000122">
    <tspan y="140.32494" x="75.000122">1000</tspan>
    <tspan y="124.74370" x="75.00008">−8</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="135.61554" x="52.011642">
    <tspan y="135.61554" x="52.011642">1001</tspan>
    <tspan y="121.21165" x="59.67443">−7</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="122.12755" x="32.528885">
    <tspan y="122.12755" x="32.528885">1010</tspan>
    <tspan y="111.09566" x="46.68593">−6</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="101.87311" x="19.458633">
    <tspan y="101.87311" x="19.458633">1011</tspan>
    <tspan y="95.90483" x="37.97243">−5</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="78.003998" x="14.828023">
    <tspan y="78.003998" x="14.828023">1100</tspan>
    <tspan y="78.00299" x="34.88535">−4</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="54.052177" x="19.3151">
    <tspan y="54.052177" x="19.3151">1101</tspan>
    <tspan y="60.03914" x="37.87674">−3</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="33.739017" x="32.285645">
    <tspan y="33.739017" x="32.285645">1110</tspan>
    <tspan y="44.80427" x="46.52377">−2</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="20.115757" x="51.705753">
    <tspan y="20.115757" x="51.705753">1111</tspan>
    <tspan y="34.58682" x="59.47051">−1</tspan>
    </text>
    <path d="m87.676,141.89a73,73,0,1,1,2.5012,-0.4862l-15.177-71.4z" transform="translate(0,5)" stroke="#000" fill="none"/>
    </svg>
    </td><td>
    <p>Visualization of two's complement numbers in binary.</p>
    <p>Zero is all zero bits. Adding and subtracting 1 makes numbers one larger or smaller, except at the transition between `01...1` and `10...0`.</p>
    <p>There is one more negative number than there are positive numbers.</p>
    <p>Two's complement is commonly used for integral values.
    Swapping sign is equivalent to flipping all bits and adding one to the result.</p>
    </td></tr></tbody>
    </table>

Biased
:   This version picks a magical fixed number (typically half of the maximum number we can write, rounded down) and calls it "the bias."
    For our numbers (`0000` through `9999`) that bias would be `4999`.
    We then define the meaning of number *x* to be the value of *x* − *bias*.
    Thus `5006` means 5006 − 4999 = seven;
    `4992` means 4992 − 4999 = negative seven.

    Biased numbers are nice in that addition and subtraction and comparison operators (`<` and `>` and their friends) work exactly like they do for unsigned numbers.
    However, multiplication and division are messier
    and humans generally find them confusing to read.

    Biased numbers are used by hardware to represent the exponent of floating-point numbers stored in binary scientific notation.

    <table><tbody><tr><td>
    <svg xmlns="http://www.w3.org/2000/svg" version="1.1" viewBox="0 0 150 150" style="width:18.5em">
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="15.324745" x="75.000122">
    <tspan y="15.324745" x="75.000122">0000</tspan>
    <tspan y="30.99356" x="75.00008">−7</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="20.047983" x="97.967705">
    <tspan y="20.047983" x="97.967705">0001</tspan>
    <tspan y="34.53599" x="90.31180">−6</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="33.567291" x="117.4518">
    <tspan y="33.567291" x="117.4518">0010</tspan>
    <tspan y="44.67547" x="103.30120">−5</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="53.824379" x="130.48605">
    <tspan y="53.824379" x="130.48605">0011</tspan>
    <tspan y="59.86829" x="111.99070">−4</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="77.824837" x="135.00021">
    <tspan y="77.824837" x="135.00021">0100</tspan>
    <tspan y="77.86863" x="115.00014">−3</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="101.70831" x="130.43951">
    <tspan y="101.70831" x="130.43951">0101</tspan>
    <tspan y="95.78123" x="111.95967">−2</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="121.95574" x="117.4518">
    <tspan y="121.95574" x="117.4518">0110</tspan>
    <tspan y="110.96680" x="103.30120">−1</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="135.48457" x="98.014236">
    <tspan y="135.48457" x="98.014236">0111</tspan>
    <tspan y="121.11342" x="90.34282">0</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="140.32494" x="75.000122">
    <tspan y="140.32494" x="75.000122">1000</tspan>
    <tspan y="124.74370" x="75.00008">+1</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="135.61554" x="52.011642">
    <tspan y="135.61554" x="52.011642">1001</tspan>
    <tspan y="121.21165" x="59.67443">+2</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="122.12755" x="32.528885">
    <tspan y="122.12755" x="32.528885">1010</tspan>
    <tspan y="111.09566" x="46.68593">+3</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="101.87311" x="19.458633">
    <tspan y="101.87311" x="19.458633">1011</tspan>
    <tspan y="95.90483" x="37.97243">+4</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="78.003998" x="14.828023">
    <tspan y="78.003998" x="14.828023">1100</tspan>
    <tspan y="78.00299" x="34.88535">+5</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="54.052177" x="19.3151">
    <tspan y="54.052177" x="19.3151">1101</tspan>
    <tspan y="60.03914" x="37.87674">+6</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="33.739017" x="32.285645">
    <tspan y="33.739017" x="32.285645">1110</tspan>
    <tspan y="44.80427" x="46.52377">+7</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="20.115757" x="51.705753">
    <tspan y="20.115757" x="51.705753">1111</tspan>
    <tspan y="34.58682" x="59.47051">+8</tspan>
    </text>
    <path d="m62.324-1.891a73,73,0,1,1,-2.5012,0.48619l15.177,71.405z" transform="translate(0,5)" stroke="#000" fill="none"/>
    </svg>
    </td><td>
    <p>Visualization of biased numbers in binary.</p>
    <p>Zero is `01...1`. Adding and subtracting 1 makes numbers one larger or smaller, except at the transition between `1...1` and `0...0`.</p>
    <p>There is one more positive number than there are negative numbers.</p>
    <p>Biased numbers are commonly used for the exponent of floating-point numbers, but not for many other purposes.</p>
    </td></tr></tbody>
    </table>

Sign bit
:   This is the version we all learned in grade school.
    Negative seven is written exactly like 7, but with a special symbol in the most-significant digits place:
    `-7`.
    With our fixed-width constraint, we'd write that as `-007` or `+007`.

    The main downside to sign-bit values is that they have two zeros: `-000` and `+000`.
    These two are written differently but have the same numerical value, so should `-000 == +000` be `true` or `false`?
    We can make any decision we want, of course, but ambiguity is not popular.

    In binary, the negative sign is generally a `1` in the first bit, while a `0` there is the equivalent of a positive sign instead.
    Sign bits are used in floating-point numbers.

    <table><tbody><tr><td>
    <svg xmlns="http://www.w3.org/2000/svg" version="1.1" viewBox="0 0 150 150" style="width:18.5em">
    <path d="m87.676,141.89a73,73,0,1,1,-27.8528,-143.32381z" transform="translate(0,5)" stroke="#000" fill="#f70" fill-opacity="0.5"/>
    <path d="m62.324,-1.891a73,73,0,1,1,27.8528,143.32381z" transform="translate(0,5)" stroke="#000" fill="none"/>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="15.324745" x="75.000122">
    <tspan y="15.324745" x="75.000122">0000</tspan>
    <tspan y="30.99356" x="75.00008">0</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="20.047983" x="97.967705">
    <tspan y="20.047983" x="97.967705">0001</tspan>
    <tspan y="34.53599" x="90.31180">+1</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="33.567291" x="117.4518">
    <tspan y="33.567291" x="117.4518">0010</tspan>
    <tspan y="44.67547" x="103.30120">+2</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="53.824379" x="130.48605">
    <tspan y="53.824379" x="130.48605">0011</tspan>
    <tspan y="59.86829" x="111.99070">+3</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="77.824837" x="135.00021">
    <tspan y="77.824837" x="135.00021">0100</tspan>
    <tspan y="77.86863" x="115.00014">+4</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="101.70831" x="130.43951">
    <tspan y="101.70831" x="130.43951">0101</tspan>
    <tspan y="95.78123" x="111.95967">+5</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="121.95574" x="117.4518">
    <tspan y="121.95574" x="117.4518">0110</tspan>
    <tspan y="110.96680" x="103.30120">+6</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="135.48457" x="98.014236">
    <tspan y="135.48457" x="98.014236">0111</tspan>
    <tspan y="121.11342" x="90.34282">+7</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="140.32494" x="75.000122">
    <tspan y="140.32494" x="75.000122">1000</tspan>
    <tspan y="124.74370" x="75.00008">−0</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="135.61554" x="52.011642">
    <tspan y="135.61554" x="52.011642">1001</tspan>
    <tspan y="121.21165" x="59.67443">−1</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="122.12755" x="32.528885">
    <tspan y="122.12755" x="32.528885">1010</tspan>
    <tspan y="111.09566" x="46.68593">−2</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="101.87311" x="19.458633">
    <tspan y="101.87311" x="19.458633">1011</tspan>
    <tspan y="95.90483" x="37.97243">−3</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="78.003998" x="14.828023">
    <tspan y="78.003998" x="14.828023">1100</tspan>
    <tspan y="78.00299" x="34.88535">−4</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="54.052177" x="19.3151">
    <tspan y="54.052177" x="19.3151">1101</tspan>
    <tspan y="60.03914" x="37.87674">−5</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="33.739017" x="32.285645">
    <tspan y="33.739017" x="32.285645">1110</tspan>
    <tspan y="44.80427" x="46.52377">−6</tspan>
    </text>
    <text style="text-anchor:middle;text-align:center;" font-size="8px" y="20.115757" x="51.705753">
    <tspan y="20.115757" x="51.705753">1111</tspan>
    <tspan y="34.58682" x="59.47051">−7</tspan>
    </text>
    </svg>
    </td><td>
    <p>Visualization of sign-bit numbers in binary.</p>
    <p>There are two zeros and two discontinuities.</p>
    <p>The negative numbers move backwards, increasing in value when the unsigned representation decreases.</p>
    <p>Sign-bit integers, as displayed here, are not used in any common hardware;
    sign bits are more common for floating-point numbers instead.</p>
    </td></tr></tbody>
    </table>

One's Complement
:   There is also a representation called "Ones' complement"
    that is often taught in courses like this
    but that is not used by common hardware today.

{.exercise ...}
Fill in the rest of the following table.
Assume you are using 6-bit numbers.
Answers are in footnotes.

Decimal | Two's-C | Biassed
|--:|--:|--:|
5 | 000101 | 100100
−5 | 111011 | 011010
11 | <input type="text" style="width:8ex"/>^[001011] | <input type="text" style="width:8ex"/>^[101010]
−1 | <input type="text" style="width:8ex"/>^[111111] | <input type="text" style="width:8ex"/>^[011110]
<input type="text" style="width:8ex"/>^[−13] | 110011 | <input type="text" style="width:8ex"/>^[010010]
<input type="text" style="width:8ex"/>^[31] | 011111 | <input type="text" style="width:8ex"/>^[111110]
<input type="text" style="width:8ex"/>^[16] | <input type="text" style="width:8ex"/>^[010000] | 101111
<input type="text" style="width:8ex"/>^[−15] | <input type="text" style="width:8ex"/>^[110001] | 010000
{/}

## On non-integral numbers

At one time there were many alternative representations of non-integral numbers,
but IEEE standards 754 and 854 define a particular representation
that has received very widespread implementation.
Numbers in this and related formats are called IEEE-style floating-point numbers
or simply "floating-point numbers" of "floats".

A floating-point number consists of three parts:

1.  A sign bit; 0 for positive, 1 for negative.
    Always present, always in the highest-order bit's place, always a single bit.

2.  An exponent, represented as a biased integer.
    Always appears between the sign bit and the fraction.

    If the exponent is either all 1 bits or all 0 bits, it means the number being represented is unusual and the normal rules for floating-point numbers do not apply.

3.  A fraction, represented as a sequence of bits.
    Always occupies the low-order bits of the number.
    If the exponent suggested this was not the normal-case number, may have special meaning.

There are four cases for a floating-point number:

Normalized
:   The exponent bits are neither all 0 nor all 1.

    The number represented by `s eeee ffff` is ± 1.ffff × 2^eeee\ −\ bias^.
    The value 1.ffff is called "the mantissa".

Denormalized
:   The exponent bits are all 0.

    The number represented by `s 0000 ffff` is ± 0.ffff × 2^1\ −\ bias^.
    The value 0.ffff is called "the mantissa".
    Note that the exponent used is "1 − bias" not "0 − bias".

Infinity
:   The exponent bits are all 1 and the fraction bits are all 0.

    The meaning of `s 1111 0000` is ±∞.

Not a Number
:   The exponent bits are all 1 and the fraction bits are not all 0.

    The value `s 1111 ffff` is Not a Number, or NaN.
    There is meaning to the fraction bits, but we will not explore it in this course.

{.exercise ...}
Complete the following table.  Assume 8-bit floating-point numbers with 3 fraction bits.
Answers are given in footnotes.

Fraction | Binary | Binary Scientific | Bits
|:-:|:-:|:-:|:-:|
7 / 8 | 0.111 | 1.11 × 2^−1^ | 0 0110 110
−∞ |  |  | 1 1111 000
1 | <input type="text" style="width:8ex"/>^[1] | <input type="text" style="width:8ex"/> × 2<sup><input type="text" style="width:4ex"/></sup>^[1.000 × 2<sup>0</sup>] | <input type="text" style="width:2ex"/> <input type="text" style="width:5ex"/> <input type="text" style="width:4ex"/>^[0 0111 000]
<input type="text" style="width:8ex"/>^[−240] | −11110000 | <input type="text" style="width:8ex"/> × 2<sup><input type="text" style="width:4ex"/></sup>^[−1.111 × 2<sup>7</sup>] | <input type="text" style="width:2ex"/> <input type="text" style="width:5ex"/> <input type="text" style="width:4ex"/>^[1 1110 111]
<input type="text" style="width:8ex"/>^[1 / 64] | <input type="text" style="width:8ex"/>^[0.000001] | <input type="text" style="width:8ex"/> × 2<sup><input type="text" style="width:4ex"/></sup>^[1.000 × 2<sup>−6</sup>] | 0 0001 000
<input type="text" style="width:8ex"/>^[7 / 512] | <input type="text" style="width:8ex"/>^[0.000000111] | <input type="text" style="width:8ex"/> × 2<sup><input type="text" style="width:4ex"/></sup>^[1.11 × 2<sup>−7</sup>] | 0 0000 111
<input type="text" style="width:8ex"/>^[1 / 512] | <input type="text" style="width:8ex"/>^[0.000000001] | 1 × 2<sup>−9</sup> | <input type="text" style="width:2ex"/> <input type="text" style="width:5ex"/> <input type="text" style="width:4ex"/>^[0 0000 001]
<input type="text" style="width:8ex"/>^[240] | <input type="text" style="width:8ex"/>^[11110000] | <input type="text" style="width:8ex"/> × 2<sup><input type="text" style="width:4ex"/></sup>^[1.111 × 2<sup>7</sup>] | 0 1110 111
<input type="text" style="width:8ex"/>^[0] | <input type="text" style="width:8ex"/>^[0] | <input type="text" style="width:8ex"/> × 2<sup><input type="text" style="width:4ex"/></sup>^[0.000 × 2<sup>−6</sup>] | 1 0000 000

{/}

## Numbering everything else

Not all data we care about is numeric.
There are both non-numeric "primitive" types like characters
and "aggregate" types made up of two or more values of other types.

When we want to introduce a primitive type,
we need to define a sufficiently large number of bits
and then create a mapping between each value of that type
and a particular bit sequence.
Since every bit sequence can be thought of as representing a number,
we could call this process "enumeration,"
but there does not need to be any obvious structure or meaning
to how the bit patterns or their number meanings relate to the represented concept.

### Character encoding

Text is pervasive in today's world, and is thus often represented in binary.
However, there is no one right way to represent characters as bits.
Indeed, it is not easy to even decide what is a character and what is not;
Unicode, one of the most extensive of the common character enumerations,
has decided that R, Ｒ, ℛ, ℜ, and ℝ, are all different characters but that
<span style="font-family:serif">R</span>,
<span style="font-family:sans-serif">R</span>,
<tt>R</tt>,
*R*, and **R** are all the same character.

Character encodings are complicated in part because there are so many of them and they have so many internal rules and special cases.
However, each encodings is just a big table of associations
like "The Angstrom sign (Å) is character number eight-thousand four-hundred ninety-one,
or 10000100101011 in binary."
Designing a character encoding is a lot of work;
detecting which one is being used can be tricky;
and some use different numbers of bits for different characters, which adds an additional level of programming complexity.
That said, most uses of characters can simply rely on some other piece of code
to keep track of all the various decisions made by the encoding designers.

### Custom encodings

It is common for programmers to wish to represent a set of primitive values other than numbers and characters.
For example, OpenGL decided that it wanted a type to represent "kind of shape"
and gave it ten values:
a group of points is `0`, a group of line segments is `1`, a ring of contiguous line segments is `2`, an open path of contiguous line segments is `3`, and so on.

These kinds of custom encodings are very common, and are generally called "enumerations" or "enums."
We'll see more on how to create and use enums later in this course.

# Which comes first?

Processors handle numbers in binary.
A binary number has a "high-order" bit and a "low-order" bit, with many bits in between.
It does not intrinsically have a "first" or "last" bit;
Sometimes we think of the high-order bit as "first" (for example, when we write it in English, we write the high-order bit first) and sometimes as "last" (for example, when we do math we get to the high-order bit last).
Fortunately, we don't need to care about the first-last distinction
because the processor doesn't interact with numbers in that way.

In general, computer memory handles numbers in bytes, base-256.
A multi-byte number has a "high-order" byte and a "low-order" byte, generally with several bytes in between.
Again, "first" and "last" are not intrinsic when describing these numbers.
However, memory does represent each byte as having a location,
like an index into a list or array.
As such, the processor has to decide, when breaking a larger number into bytes,
whether it is going to put the high-order or low-order byte in the first spot in memory.
Other parts of computers, such as disks and network drivers, also need to be told bytes in a first-to-last order.

Processors (and people) are not consistent in what decision they make.
Some, like x86, x86-64, addition, and Arabic numerals in the original Arabic, put the low-order byte first and are called "little-endian".
Others, like MIPS, PowerPC, comparison, and Arabic numerals in most European languages, put the low-order byte last and are called "big-endian".
A few, like ARM and students in classes like this, understand both and can be configured to act in either a big-endian or little-endian mode.

{.aside ...} In English (and most other languages today) we use base-10 and Arabic numerals; thus thirty-five is written 35.  The high-order digit is on the left, the low-order digit is on the right.  When you put Arabic numbers inside of English text it thus looks big-endian because English is read left-to-right.  However, when you put the *same* number inside of Arabic text it looks little-endian because Arabic is read right-to-left.
{/}

It is important to note that endianness applies only when breaking a primitive value into pieces and putting those pieces in some arbitrary order.
Endianness does not apply inside the processor, where numbers are stored in whole without an arbitrary order.
Endianness also does not apply to naturally-ordered values like the elements of a list,
as these are placed in their natural order on both big- and little-endian computers.

{.example ...}
Consider an array (a list stored contiguously in memory)
containing two 16-bit numbers, \[0x1234, 0x5678\].
If this array were placed in memory at address 0x108, we'd find the following in memory:

 address    little-endian    big-endian
---------- ---------------  ------------
 108          34              12
 109          12              34
 10a          78              56
 10b          56              78

Observe that the order of bytes *within* each number changes based on endianness,
but that all of the following are unimpacted by endianness:

- Each number occupies two bytes.
- The address of a number is the address of its "first" byte, whichever byte that is in the endianness being used.
- The elements of an array are in the same order in memory as they are in code.
- The second element's address is 2 bytes past the first, at 0x10a.
- It is bytes, not nibbles or bits, that are placed in memory, so we still see byte `0x34` (`0b00110100`) not nibble-reversed `0x43` (`0b01000011`) nor bit-reversed `0x2c` (`0b00101100`).
{/}

# Intentional redundancy

Although digital systems are much less sensitive to noise than are analog systems,
sometimes noise still does impact the signal, and when it does
the consequences are potentially quite large, particularly if the noise flips a high-order bit.
In part because of this, it is common to include intentional redundancy in digital signals.
By including information that could have been fully determined without its presence,
it becomes possible to compare that information with what would normally have been generated
and use that to see if the information has arrived in its intended form.

## Detection versus correction

The simplest form of redundancy to design allows you to detect if a group of bits has been modified in small ways.
The "small ways" caveat is important: there is no form of redundancy that is proof against large changes to the information,
as the redundant data itself is encoded in bits and if those bits are changed to match the changes to other bits, the data will look consistent.

A single extra bit can be enough to detect that a bit got flipped due to noise.
To be able to automatically correct that flipped bit in an *n*-bit signal requires at least log~2~(*n*) extra bits, as anything less than that has insufficient information content to identify which bit needs correction.

{.exercise ...}
Measure the detection and correction ability of redundancy in a human language of your choice.

1. Find a large corpus of example text (the larger the better)
2. Write a program that repeatedly
    1. selects a random substring from the corpus, long enough to give a little context (perhaps 2 dozen words)
    2. randomly either (a) leave it alone or (b) change one word in the central half of what you show
    2. prompt the user to either identify the text as unchanged or changed, and if changed to identify what word was changed and what it was supposed to be
    3. track the number of correct and incorrect identifications and corrections
{/}

While correction may seem desirable, often the easier detection criteria is sufficient
because in many cases the recipient of detected-bad data can simply request the sender to send it again, repeating as needed until a good copy is obtained.


## Parity, checksums, error-correction codes, and digests

There are many different techniques used to add structured redundancy to sets of bits.
This section describes just a few.

Parity

:   The "parity" of a set of bits is `0` if an even number of the bits are `1`s and `1` if an odd number of the bits are `1`.
    Adding a parity bit (sometimes called a "check bit") to a number is a simple way to make single-bit errors detectable.

{.exercise ...}
The following table shows 1-byte values and their check bit.
Identify which ones have a parity error.

Byte | Check | Has parity error
|:--:|:--:|:--:|
00000000 | 0 | <input type="checkbox" disabled>
01001101 | 1 | <input type="checkbox" checked disabled>
11111000 | 0 | <input type="checkbox">
10101010 | 0 | <input type="checkbox">
00011100 | 1 | <input type="checkbox">
01000010 | 1 | <input type="checkbox">
11111111 | 1 | <input type="checkbox">

Answers are in this footnote^[error, correct, correct, error, error]
{/}  

Multiple parity
:   If you store the parity of multiple groups of bits, you can use the intersections of the groups that have errors to determine which bit had the wrong value and correct it.
    For example, given <tt>11001010</tt> we could compute the following parity bits:

    Partity of      is
    ------------  -------
    `11001010`     `0`
    `1100    `     `0`
    `11  10  `     `1`
    `1 0 1 1 `     `1`

    Changing any input bit will change a unique set of parity bits;
    for example, changing the second-most-significant bit will change the first three parity bits but not the fourth.

    This is an example of how one could begin to design a correcting code,
    but it is not fully usable as is because an error in the parity bits themselves
    cannot be unambiguously identified and corrected.
    Full correctable redundancy is more complicated to design;
    common designs include convolution codes (based on Boolean polynomials)
    and Hamming codes (based on parity bits, but selected more carefully than the example above).

CRC
:   Many digital formats make use of a family of error detection algorithms
    collectively known as Cyclic Redundancy Checks or CRC.
    The IEEE has standardized (in IEEE 802-3) one, commonly called CRC32,
    that is used in common formats such as [the PNG standard](//www.w3.org/TR/PNG/#5CRC-algorithm).

    CRC's are cyclic in the sense that they can be applied to any length input
    by repeating a simple procedure.
    This makes them one type of **digest**: an algorithm that accepts long inputs
    and produces small summary outputs.
    CRCs are similar to the other most common form of digest (called a hash),
    but are generally optimized for speed and for detecting bit-level changes.
    Other digests are used in security or memory allocation contexts
    and have different design goals.

Checksum
:   "Checksum" is a generic term for any easily-defined, easily-computed value that depends on a larger message.
    Parity, CRC, hashes, and many others are all examples of checksums.
    The word itself suggests another simple approach:
    treat each byte as a number and sum them all up.

    There is not just one kind of checksum.
    Often when you see documentation refer to a "checksum"
    they either mean that they have a custom way of producing the error-detecting redundancy
    or that the specific method used is unimportant in the context of the documented features.


Although error detection and correction through redundant data is important for the success of all kinds of digital information,
and commonly implemented in everything from computer memory chips to archive file formats,
it is relatively uncommon for a programmer to need to think about checksums in detail.
It is usually enough to know that messages may contain some error-checking information
and that recipients may inform providers that a message arrived in a corrupted state.
