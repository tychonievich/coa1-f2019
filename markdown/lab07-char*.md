---
title: char *coding
...

This lab could have been a PA, but we thought it would be more useful for you to be able to work together and share ideas and solutions.
We strongly encourage teamwork in this lab.

This lab will run as follows:

1. Review `char *` strings
2. Implement a variant of `strlen`
3. Compare your implementation with that of another student
4. Find a way to make `strlen` shorter/simpler, and note what you did in a comment
5. Implement a variant of `strtok`
6. Compare your implementation with that of another student
7. Find a way to make `strtok` shorter/simpler, and note what you did in a comment
8. Show a TA how you made each function smaller

# String Overview

## Layout

In C, strings are pointers to arrays of characters, or `char *`s.
They do not include length information; instead, they use the special value 0 to note the end of the string.
Thus, if memory contains

+-------+--+--+--+--+--+--+--+---+--+--+--+--+--+--+--+---+
|Address|20|21|22|23|24|25|26|27 |28|29|2A|2B|2C|2D|2E|2F |
+-------+--+--+--+--+--+--+--+---+--+--+--+--+--+--+--+---+
|`char` |w |e |l |c |o |m |e |\\0|t |o |  |c |h |a |r |\\0|
+-------+--+--+--+--+--+--+--+---+--+--+--+--+--+--+--+---+

Then if `char *x` contains `0x20`, it looks like the string `"welcome"`
and if it contains `0x28`, it looks like the string `"to char"`.
Note that there is no magic to starting at the beginning of something;
if `x` contains `0x22` then it looks like the strong `"lcome"`
and if it is `0x27` it looks like the empty string `""`.

## Const

String literals are generally placed by the compiler in read-only memory,
and have the type `const char *` not just `char *`.
Thus,

````c
const char *c = "hello";
c[1] = 'u'; /* error: cannot modify read-only memory */
c = c + 2;  /* OK: c can change but, the memory it points to cannot */
````

If you want to get a mutable copy of a const string, you can use `strdup`.
From `man strdup` we find

    #include <string.h>
    char *strdup(const char *s);

    The  strdup()  function  returns  a pointer to a new string which is a duplicate of the string s.  Memory for the new string is obtained with malloc, and can be freed with free.

This means you'd use is as

````c
const char *s1 = "hello"; /* initialized in read-only memory */
char *s2 = strdup(s1);    /* make a copy in mutable memory   */
s2[1] = 'u';              /* change one letter of the copy   */
puts(s2);                 /* display the altered copy        */
free(s2);                 /* don't litter: discard the copy  */
````

Note that the `free` when you are done is somewhat important,
but won't break your code if you leave it off in this lab.
We'll discuss more about `free` in class later this week.

## Display

The simplest way to display a string in C is using the function `puts`.
An extract from the manual page `man puts` says

    #include <stdio.h>
    int puts(const char *s);

    puts() writes the string s and a trailing newline to stdout.
    puts() returns a nonnegative number on success, or EOF on error.

Thus

````c
const char *s = "welcome\0to char";
puts(s);
puts(s+2);
puts(s+7);
puts(s+8);
````


will display

    welcome
    lcome
    to char

# Writing string code

Let's start with some basic string functions.
These are in the standard C library, but are worth implementing by hand to better understand them.

## `strlen`

The `strlen` function is described in `man strlen` as

    size_t strlen(const char *s);

   The strlen() function calculates the length of the string s, excluding the terminating null byte ('\0').

Write an implementation of this function, naming it `mystrlen` instead of `strlen`.

{.example ...}
The following code

````c
const char *s = "even elephants exfoliate";
size_t slen = mystrlen(s);
puts(s + (slen/2));
````

should display
    
    ts exfoliate

{/}


After you've written an implementation, compare with another student.
Work together to make a simpler version (you'll need to explain what you did to a TA later, so remember your changes).

There exist very short solutions to this; I know of one where the entire function body has just 3 statements and 31 non-white characters...


## Simplified `strtok`

Two library functions, `strsep` and `strtok`, both implement a string-splitting behavior.
You will implement a simplified version

`char *simple_split(char *s, char delim);`
:   Given a string and a delimiter character, do the following:
    
    - if `s` is `NULL` or `s[0]` is `\0`, return `NULL`
    - find the first `delim` character in `s`
        - if there is none, return `NULL`
        - otherwise, replace it with `\0` and return a pointer to the character after it

{.example ...}
The following code

````c
char *s = strdup("can all aardvarks quaff?");
char *bit = simple_split(s, 'a');
puts(s);
puts(bit);
free(s);
````

should display
    
    c
    n all aardvarks quaff

{/}

{.example ...}
The following code

````c
char *trash, *bit, *s;
trash = bit = s = strdup("can all aardvarks quaff?");
do {
    s = bit;
    bit = simple_split(s, 'a');
    puts(s);
} while(bit);
free(trash);
````

should display

    c
    n 
    ll 
    
    rdv
    rks qu
    ff?

{/}


After you've written an implementation, compare with another student.
Work together to make a simpler version (you'll need to explain what you did to a TA later, so remember your changes).

There exist very short solutions to this; I know of one where the entire function body has just 5 statements and 70 non-white characters...

# Pass off

Show a TA your implementations and describe how you made them smaller (or why you beleive they are as small as they can be).
