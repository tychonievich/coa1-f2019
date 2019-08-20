---
title: C standard library manual pages
...

This is a guide to understanding manual pages and common C standard library conventions.

# `man man`

The `man` command can be run by typing `man something` on the command line.
If you type `man man` you'll get the manual page about the `man` command.
It gives guidance to how `man` pages are organized, and it is worth reading the **Description** section of `man man` in full.
A few useful commands to remember:

--------------------------------------------------------------------------------
Command                         Why to use it
------------------------------- ------------------------------------------------
`man 3 functionname`            Section 3 has library functions; adding `3` can
                                help if the same name is also a command-line
                                tool

`man -f functionname`           This **f**inds pages that discuss `functionname`
                                displaying the page name, section, and a short
                                synopsis of each.

`man -k some_text`              This treats `some_text` as a regular expression
                                and searches for it within man pages,
                                displaying the page name, section, and a short
                                synopsis of each.
--------------------------------------------------------------------------------

Within `man`, you can use several keyboard shortcuts to help navigate the page.
`h` displays the full key help, and `q` quits the man page viewer.
A few of the most used commands include:

Moving
:   The arrow keys, page-up and page-down, home and end move through the page

Searching
:   If you type `/`, you enter search mode.
    You can then type a regular expression to look for.
    Enter locks in the regular expression as the current search pattern and finds the first one one or below the current screen.
    `n` then goes to the next match, repeated until the end of the file is reached.
    `N` searches above the current screen instead.

Manual pages use various formatting to help provide contextual cues;
a few to know include:

--------------------------------------------------------------------------------
Concept                         Displayed as
------------------------------- ------------------------------------------------
Another manual page             **printf**(3): the page name in bold,
                                the manual section in parentheses

Function names                  **printf**(): the name in bold,
                                with empty parentheses to mean "a function"

Adjustable variable names       <u>stream</u>: underlined
--------------------------------------------------------------------------------



# Anatomy of a library function manual page

Each manual page has several headers:

Name
:   the set of manual page names that are all shared by this page.
    Often several related functions are documented together

Synopsis
:   The relevant C header file `#include`{.c}s,
    and the function prototypes defined in them that are described on this page.

Description
:   A description of how this function or set of functions operate,
    including anything they assume about their arguments,
    any rules that apply to their return values, etc.
    
    The description is often long enough to have multiple subsections with their own headers.
    In these cases it is often not feasible to put them in the single correct order,
    so you may need to jump about a bit to find what you are looking for.

Return Value
:   Some functions describe their return value in their *Description*,
    but many have a special section for it.
    
    See also [Reporting errors] below for some common trends in return values.

Conforming To
:   Various standards bodies help ensure that computers can inter-operate.
    You can probably ignore this section until you gain several years more experience with C.

Notes
:   If present, this often points out common mistakes
    or limitations of the functions.
    Always read it.

Bugs
:   If present, this points out limitations of the functions that can lead to them harming your program if misused.
    *Always* read it.

Example
:   Often the first place to look when reading a new manual page.
    They vary in detail, but can be a great place to learn the scope of what the function can (and cannot) do.

See Also
:   If the function you looked up isn't exactly what you want,
    other functions to consider may be listed here.


# Common function conventions

While there is no rule that the following need to be observed,
many functions in the C standard library use the following conventions.

## Results in parameters

If a function should return more than a single primitive value,
many functions will instead *return* and error-checking integer (see [Reporting errors])
and use a pointer-type argument to provide the "returned" value.
Some will mix and match, both returning and using a pointer parameter to provide information.

{.example ...}
Consider the function `char *strsep(char **stringp, const char *delim)`{.c}.
It returns a `char *`, which is a single token of a string after it is split on `delim`.
But the input string, instead of being a normal `char *` is a `char **`.
this is because it modifies the argument to point to the string following the split.

Inside this function, there is something like the following:

````c
*stringp += length_of_first_token;
````

Because this modifies not the local variable `stringp` but rather the pointer it points to,
the calling function can see that pointed-to pointer change.
{/}

## Reporting errors

Many functions return an error-checking result of some kind.
Often this involves returning an `int` where some numbers mean "success!" and others "failure".

When a function has a single "failure" return type, it often also "sets <u>errno</u>" to some specific error code.
The global variable `errno` is defined in `#include <errno.h>`{.c}, along with `#define`d names for various kinds of errors.
If you want to know that something went wrong with these functions, you have to 

1. Check if the returned value was an error value
2. If so, check `errno` to find out what the error was

A few functions instead directly return either `0` or an error number declared in `<errno.h>` directly.
A few others set `errno` with no special return value, requiring `errno` to be checked manually.

{.example ...}
The `strtol` function
converts a `char *` to the integer its (as a string) contains.
It returns the result of the conversion, *unless* the value would underflow or overflow.
If an underflow occurs, `strtol` returns `LONG_MIN`; if  an  overflow occurs, `strtol` returns `LONG_MAX`.  In both cases, `errno` is set to `ERANGE`.
Additionally, if the input was not a valid value, it sets `EINVAL`, with no officially defined return value.

Thus, a correct use would be

````c
errno = 0; // clear the error number
long ans = strtol(argv[1]);
long error = errno;
errno = 0; // clean up after ourselves
if (error != 0) {
    puts("There was some kind of conversion error");
    if (error == EINVAL)
        printf("...because \"%s\" is not a valid integer\n", argv[1]);
    else if (error == ERNAGE)
        printf("...because \"%s\" is too big to represent\n", argv[1]);
    else
        printf("...because of an unexpected error (number %d) processing \"%s\"\n", error, argv[1]);
} else { 
    // ...
}
````
{/}

## Avoiding `malloc`

It is very uncommon for library functions to use `malloc`
unless allocating memory is central to their primary purpose (as e.g. for `malloc` or `strdup`).
Instead, functions that would need to return a variable-length array of data (such as a string)
will generally require that a pointer to memory to write it to is passed in as an argument.

{.example ...}
The `read` function reads bytes from a file or file-like object.
Thus it requires three parameters:

1. The file to read from
2. A pointer to a buffer to place the read contents into
3. The maximum number of bytes it is allowed to place there

It communicates how much of the buffer it filled as a return value.

Putting these pieces together, we get `read`'s signature: `ssize_t read(int fildes, void *buf, size_t nbyte);`{.c}.
{/}

