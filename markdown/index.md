---
title: Computer Organization and Architecture 1
...

# Writeups

The following are the main writeups created for this course:

- [Boolean Algebra and Gates](bool.html) including bit-fiddling
- [Bits and Beyond](bits.html) including number representations and a little information theory
- [Components of digital computers](parts.html), including voltage, registers, and HDLs
- [Designing a processor](isa.html), including von Neumann architecture, ISAs, condition codes, and RISC/CISC
- [x86-64 Summary](x86.html), including calling conventions
- [C, a guide and reference](c.html), including the C preprocessor
- [An overview of memory](memory.html), including garbage detection and common memory errors
- [C standard library manual pages](manpage.html), including a guide to using the `man` command
- [C++ Inheritance](vtable.html), including how vtables enable virtual functions

The following are additional references:

- [Using SSH](help-ssh.html), including how to set up passwordless authentication
- [Command-fu](command-fu.html), a few simple command-line examples (unfinished)
- [Debugger example](cmdadd.html), using `lldb` and `ghex`
- [Tips for Linux on your own computer](linux.html), including how to connect to UVA's instantiation of eduroam and handling digital certificates.

# Overview 

This is part of a pilot of a new set of CS courses.

## Eligibility

You should take this course if and only if

1. You have credit (or passed the placement test) for at least one of CS 1110, CS 1111, CS 1112, CS 1113, or CS 1120
1. You have do **not** have credit for CS 2110 or beyond
1. You are, or are planning on becoming, a BA CS or BS CS major^[At present, the pilot is not approved for those planning on CpE, or seeking a CS minor.]
1. You are able to enroll in **all three** of COA1 (CS 2501-200), DSA1 (CS 2501-100), and Discrete Math^[Discrete Math is initially limited to declared majors; [that changes](https://goo.gl/tTDUqf) after the initial enrollments pass] (CS 2102) this Fall^[Already have credit for Discrete Math? Then you just need DSA1 and COA1.]
1. You will be on campus next Spring and able to take COA2 and DSA2

More information about the pilot of these courses and reasoning for these limitations may be found at <http://pilot.cs.virginia.edu/>.

## Improvements

This is a second round pilot offering of one of several new courses. It addresses several concerns of the faculty, including both reordering material for a better flow through the curriculum and laying a better foundation for advanced coursework. Additionally, as a pilot, it offers a smaller, more intimate experience with a common cohort of peers than our usual very-large classes.

## Scope and Content

In this course, we 

- Begin with how data can be stored as charges in silicon and work up
    - In hardware design, through gates and registers to general-purpose computers.
    - In data representation, through bits and bytes to records, arrays, and pointers.
    - In process representation, through circuits and assembly to C.
- Learn basic command-line tools and accessing command-line documentation.
- Practice quite a bit of C coding and using the C standard library.
- Discuss how security and social topics are related to these ideas.

For the sake of conversing with those familiar with our other course offerings,
this course covers the assembly-and-C half of CS 2150 "Program and Data Representation";
the basics of ECE 2330 "Digital Logic Design";
and the first part of CS 3330 "Computer Architecture";
in addition to having several new topics we felt were under-represented in our
previous set of course offerings.
