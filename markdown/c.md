---
title: C, a guide and reference
...

This is intended to be a practical guide (rather than an authoritative guide) to C,
as implemented by clang and gcc for the x86-64 processor family.

# Data Types

The `sizeof(...)` operator returns the size of a type in bytes.
Thus, `sizeof(int)`{.c} is `4`, not `32`.

## Primitive

### Integer

The integer data types are

name            bits        representation              notes
--------------- ----------- --------------------------  ------------------------------------
`_Bool`         1 or more   undefined                   rarely used; for all types, `0` is false, anything else is true
`char`          8           signedness undefined        usually used for characters, sometimes for bytes
`signed char`   8           2's complement
`unsigned char` 8           unsigned integer
`short`         16          2's complement
`int`           32          2's complement
`long`          32 or 64    2's complement              32 bits if compiled in 32-bit mode; for 64-bit, add the `-m64` flag when compiling
`long long`     64          2's complement

Each has an `unsigned` version (e.g., `unsigned short`, etc). If `unsigned` is used as a type by itself, it means `unsigned int`.

Integer literals will be implicitly cast to the correct type upon assignment;
thus `char x = -3`{.c} will turn `-3` into an 8-bit value automatically,
as `int x = 'x'`{.c} will turn `'x'` into a 32-bit value.
This only works up to int-sized literals.

To force a literal to be long add a `l` or `L` to the end; to force it to be unsigned add a `u` or `U`.
This is generally only needed for very large constants, like `unsigned long very_big = 9223372036854775808ul`{.c}.

Character literals are integer literals written with a different syntax.
There is no significant difference between `'0'` and `48` other than legibility.

### Floating-point

The floating-point datatypes are

name            exponent bits   fraction bits   total size              literal syntax
--------------- --------------- --------------- ---------------------   ------------------------------------
`float`         8               23              32 bits (4 bytes)       `3.1415f` -- `f` or `F` for `float`
`double`        11              52              64 bits (8 bytes)       `3.1415`  -- no suffix
`long double`   15              64              80 bits (10 bytes)      `3.1415l` -- `l` or `L` for `long`

Note that `long double` has traditionally only differed from `double` on x86 architectures.

### Enumerations

The `enum` keyword is a special way of defining named integer constants,
typically in ascending order unless otherwise specified.

````c
enum colors { a, b, c, d=100, e };
/* a is 0, b is 1, c is 2, d is 100, and e is 101 */

int f = e; /* equivalent to f = 101 */
````

### void and casting

There is also a special `void` type that means either "a byte with no known meaning" (if used as part of a pointer type) or "nothing at all" (if used as a return type or parameter list).

Casting between integer types truncates (if going smaller) or zero- or sign-extends (if going larger, depending on the signedness of the value) to fit the available space.
Casting to or from floating-point types converts to a nearby^[Oddly, not always *the* nearest value; floating-point numbers use a "round to even" rule that sometimes rounds in a different direction than you expect in order to get the last bit of the fraction to be a `0`.] representable value (which may be infinity),
with the exception that casting from float to int truncates the reminder instead of rounding.

## Pointers

For every type, there is a type for a pointer to a value of that type.
These are written with a `*` after the type:

````c
int *x;     /* points to an int */
char *s;    /* points to a char */
float **w;  /* points to a pointer that points to a float */
float ***a; /* points to a pointer that points to a pointer that points to a float */
````

A pointer to any value stored in memory can be taken by using the address-of operator `&`
Thus `&x`{.c} is the address of the value stored in `x`,
but `&3`{.c} is an error because 3 is a literal and does not have an address.
You also can't take the address of the result of an expression: `&(x + y)` or `&&x` are both errors as well.

You de-reference pointers with the same syntax used to create them: a `*` before the variable.

````c
int *x = &z;     /* x = pointer to z */
int x1 = *x;     /* x1 == z */
````

You can also de-reference pointers with subscript notation; `*x` and `x[0]` are entirely equivalent,
as are `*(x + n)` and `x[n]`.

There is syntactic ambiguity when combining `*` and `[1]`.
Is `*a[1]` the same as `*(a[1])` or `(*a)[1]`?
This is solved by operator precedence (`[]` before `*`),
but is not intuitive to most programmers
so you *should always* use parentheses in these cases.


All pointers are the same size (the size of an address in the underlying ISA) regardless of the size of what they are pointing to;
thus `sizeof(char *) == sizeof(long double *)`{.c}.
Two special int types^[Defined using `typedef` in `<types.h>`] are used to be "an integer the size of a pointer":
`size_t` is an `unsigned` integer of this size, and `ssize_t` is a `signed` integer of this size.
With the compilers and ISAs we are using this semester `size_t` is the same as `unsigned long` and `ssize_t` is the same as `long`.

When you add an integer to a pointer, the address stored in the pointer increases by a multiple of the `sizeof` the pointed-to type.

````c
int x = 10;
int *y = &x;                    // y points to x
int *z = y + 2;                 // z points 2 ints after x
long w = ((long)z) - ((long)y); // w is 8, not 2.
````



## Composite

There are two basic compound types in C: the `struct` and the array.

### Array

An array is zero or more values of the same type stored contiguously in memory.

````c
int array[1000];             /* an array of 1000 int values */
````

Except when used with `sizeof` and `&`, arrays act exactly like pointers to their first element;
notably, this means that `array[23]` does what you expect it to do: access the 24th element of the array.

The `sizeof` an array is the total bytes used by all elements of the array:


````c
unsigned x = sizeof(array);  /* 4000: sizeof(int) * 1000    */
````

The `&` an array is the `&` of its first element (i.e., `&array == &(array[0])`).


Parentheses are allowed when declaring types, although their meaning is counter-intuitive to many students:

````c
char *pc[10];     /* an array of 10 (char *)s */
````
````c
char *(pc[10]);   /* an array of 10 (char *)s */
````
````c
char (*pc)[10];   /* a pointer to an array of 10 (char)s */
````

The rule here is that we declare variables *exactly* as we would use them:
a point to an array would first be dereferenced (`(*pc)`) and then indexed (`(*pc)[i]`) to get a `char`
so we declare it as `char (*pc)[10]`.

Arrays literals use curly braces and commas.

````c
int x[10] = {1, 1, 2, 3, 5, 8, 13, 21, 34, 55};
````

Unless initialized with a literal like this, the contents of an array are *undefined* (i.e., may be any random values the compiler thinks is most efficient) when created.

Arrays cannot be resized after being created.

### Struct


A `struct` also stores values contiguously in memory,
but the values may be of different types and are accessed by name, not index.

````c
struct foo {
    long a;
    int b;
    short c;
    char d;
};           /* note the ; at the end; it is REQUIRED! */
````

The name of the resulting type includes the word `struct`

````
struct foo x;
unsigned long a = sizeof(struct foo);
x.b = 1234;
x.a = x.b - 5;
````

Compilers are free to lay out the data elements of a structure with padding between elements if they wish;
this is often done in practice to improve data alignment, so in the above example we expect `a` to have a value larger than the minimal 15 bytes needed to store those fields.

Structures are passed by value; that is, using them as arguments, return types, or with `=` means that all of their fields are copied.
This is inefficient for all by the smallest `structs`, so often pointers to structures are passed, not the structures themselves.

Because all pointers are the same size, you can have code use a pointer to a `struct`
without knowing what is inside the `struct`;
the only need to be known for `sizeof` and the `.` operator to work.

````c
struct baz;                  /* just says "a struct of this name exists"   */
void swizzle(struct baz *);  /* just says "a function of this name exists" */

/* Swizzles an array of struct bazs                           *
 * This code does not need to understand what a struct baz is */
void swozzle(struct baz **x, int n) {
    for(int i=0; i<n; i+=1) swizzle(x[i]);
}
````

Structure literals are written using curly braces and commas, optionally with `.fieldname =` prefixes

````c
struct a {
    int b;
    double c;
}

/* Both of the following initialize b to 0 and c to 1.0 */
struct a x = { 0, 1.0 };
struct a y = { .b = 0, .c = 1.0 };
````

Unless initialized with a literal like this, the values of fields of a struct are *undefined* (i.e., may be any random values the compiler thinks is most efficient) when created.


## Constant

If a type is preceded by `const`, the compiler is free to perform optimizations that assume that no code will ever change the values of this type after they are first initialized.

As a special syntax, a string literal like `"hello"`{.c} does two things:

1. It ensures there exists somewhere an array of characters `{'h', 'e', 'l', 'l', 'o', 0}`, typically in read-only memory.
    - Note the `0` at the end (that's byte-0 not character-0). This is how C knows the string is over.
2. It returns a `const char *` pointing to the `h`.

## typedef

You can give new names to any type by using the `typedef` statement:

````c
typedef int Integer;
Integer x = 23;

typedef double ** dpp;
double y0 = 12.34;
double *y1 = &y0;
dpp y = &y1;

struct foo { int x; double y; };
typedef struct foo foo;
foo z;
z.x = x;
z.y = **y;
````

`typedef` type names are aliases to the old names;
the compiler will treat both the original and new name as equivalent in all type checking.

Sometimes `typedef` is used with *anonymous* `struct`s:

````c
struct { int x; double y; } foo;
foo z;
````

## Union

A union is like a struct, except that all of the fields are stored in the same memory address.
In practice, this means only one of them has a meaningful value at a time.

````c
union odd {
    long long i;
    double d;
};

union odd x;
x.i = 0x1234;    /* x's memory now contains 34 12 00 00 00 00 00 00 */
double y = x.d;  /* y is now 2.30235e-320 (those same bytes) */

x.d = 0x1234;       /* x's memory now contains 00 00 00 00 00 34 b2 40 */
long long z = x.d;  /* z is now 0x40b2340000000000 (those same bytes) */
````

## You can do bad things

C does not try to prevent you from doing bad things.

````c
float x = 123.567f;   /* A floating-point number */
int y = *((int *)&x); /* An integer made from the same bytes as the floating-point number */

int z[4];             /* An array of 4 integers */
int w = z[254];       /* An integer made from the contents of memory 1000 bytes after the end of z */

const char *s = "hi"; /* compiler makes the string in memory the OS won't allow us to change */
char *t = (char *)s;  /* we get a pointer to that memory that C will allow us to change */
t[0] = 'H';           /* we try to change that memory (the OS will crash our program) */
````

C's general attitude is "every rule has an exception" and "the programmer knows best".
It might make you do some complicated casting to do things, but it won't stop you if you are determined.


# Control constructs

## Braces and scope

Any statement may be replaced with a sequence of statements inside braces.
Variables declared inside a set of braces vanish at the end of those braces.

````c
int x;
{
    int y;
    x = y;  /* OK, both x and y in scope */
}
y = x; /* ERROR: y is no longer in scope */
````

## Flow of control

### Nice and common ones

#### if

Any statement may be preceded by `if ( ... )`;
the statement will only be executed if the expression inside the parentheses yields a non-zero value.

Any statement following a statement preceded by `if ( ... )` may be preceded by `else`;
the statement will only be executed if the expression inside the `if`'s parentheses yields a zero value.

#### while

Any statement may be preceded by `while ( ... )`;
the statement will only be executed if the expression inside the parentheses yields a non-zero value,
and will continue to be executed until that condition stops being true.

#### for

The special construct `for (e1; e2; e3) s;`{.c}
is equivalent to the following:

````c
{
    e1
    while (e2) {
        s;
        e3;
    }
}
````

with a slight twist: if `s` contains a `continue`, it jumps to `e3` instead of to `while (e2)`.


If `e2` is omitted, it is assumed to be `1`, so `for(;;) s;` repeats `s` forever.

### Ugly and uncommon ones

#### do-while

The syntax `do s; while (e);`{.c} means the same as `s; while (e) s;`:
that is, it always does `s` once before first checking `e`.
In my experience, this is used for less than 1% of loops.

#### label and goto

Any line of code may be preceded by a label:
an identifier followed by a colon.

The `goto some_label;`{.c} statement unconditionally jumps to the code identified by that label.

In 1968 Edgar Dijkstra write an article "[Go To Statement Considered Harmful](Dijkstra68.pdf)".
Since then, the use of `goto` in code has dropped significantly; 
it's now usually a sign either of over-emphasis on optimization
or a shim to avoid having to redesign poorly-organized code.
However, there are a few situations where it can be handy, so it does sometimes show up in high-quality code.

#### switch

The `switch` statement in C may be implemented in several ways by the compiler,
but it is designed to be a good match for the "jump table" approach.

The syntax of the `switch` is as follows:

````c
switch(i) {
    case 0:
        statements;
        break;
    case 1:
        statements;
        break;
    case 3:
        statements;
        break;
    case 4:
        statements;
        break;
    default:
        statements;
}
````

Conceptually, this is

- a block of code
- with multiple labels
- where the labels are numbered, not named

and it operates like the (invalid) code

````c
c_code *targets[5] = { (case 0), (case 1), (default), (case 3), (case 4) };
if (0 <= i && i < 5) goto targets[i];
else goto default;
````

The `break` (as with a `break` in a loop) stops running the code block and goes to the first statement after it.

Many people think of a `switch` as being a nice way to write a long `if`/`else if` sequence,
and are then annoyed by its limitations and quirks:
it has to have an integer selector (as this is really an index),
and it "falls through" to the next case if there is no `break`.
Hence the following example, taken from [wikipedia](https://en.wikipedia.org/wiki/Switch_statement):

````c
switch (age) {
  case 1:  printf("You're one.");              break;
  case 2:  printf("You're two.");              break;
  case 3:  printf("You're three.");
  case 4:  printf("You're three or four.");    break;
  default: printf("You're not 1, 2, 3 or 4!");
}
````

Because many programmers make mistakes with `switch`,
it is common to see them banned by style,
or augmented with a special style,
or later languages to use a similar syntax in ways a jump table cannot handle,
or mostly C-compatible languages augmenting them with rules like
"each case bust either end with `break` or with an explicit `fallthrough`/`goto case`".

Most compilers have several different implementations of `switch` they can pick between;
they might use a jump table, a sequence of `if`/`else if`s, a binary search, etc.

# Functions

## Most common use

The most common use of functions in C looks much like you are used to from other languages:
a return type, a name, a list of typed parameters in parentheses, and a body in braces.

````c
void baz(int i, char *b, float c) {
    b[i] = (char)c;
    return;
}
````

It is also common to declare functions before defining them,
in part because C requires functions to be declared before use.

````c
int is_even(unsigned n);
int is_odd(unsigned n);

int is_even(unsigned n) {
    if (n == 0) return 1;
    else        return is_odd(n - 1);
}

int is_odd(unsigned n) {
    if (n == 0) return 0;
    else        return is_even(n - 1);
}
````

Often the declarations or **function headers** are put in a separate file,
called a **header file** and traditionally named with the suffix `.h`.
The `#include` directive can thus grab all of these at once,
simplifying coding without increasing the size of the resulting `.c` file or the compiled binary.


## Syntax variations

However, C allows several variations on this theme.

-   Function return types can be omitted, defaulting to `int`:
    
    ````c
    min(int a, int b) { return a < b ? a : b; }
    ````
    
-   Function parameter types can be omitted, defaulting to `int`:

    ````c
    min(a, b) { return a < b ? a : b; }
    ````

-   Function parameter types can be specified between the `)` and the `{`:

    ````c
    void baz(i, b, c)
    int i;
    char *b;
    float c;
    {
        b[i] = (char)c;
        return;
    }
    ````
    
    Technically, this does something called "promotion" and has various quirks;
    for this and other reasons it is often called "old-style" and generally discouraged.

-   A zero-argument function can be written as either

    ````c
    int three() {
        return 3;
    }
    ````
    
    or 
    
    ````c
    int three(void) {
        return 3;
    }
    ````

-   The `main` function (only) will return `0` if it is missing a `return`,
    and may omit its arguments upon definition.
    

## It's all convention

C passes arguments using a calling convention.
This is obeyed blindly by both the caller and the callee;
so if the caller thought the callee had different argument types than it did,
neither will notice they have a problem; they'll just silently do the wrong thing.

{.example ...}
Consider the following pair of files:

<figure><caption>baz.c</caption>
````c
long bar(char *);

/******* adds bar("hello") to its argument *******/
long baz(long x) {
    return bar("hello") + x;
}
````
</figure>

<figure><caption>bar.c</caption>
````c
/** returns the requested suffix of "ten letter" */
char *bar(long x) {
    char *c = "ten letter";
    return c + (x%10);
}
````
</figure>

When executed, 

1. `baz` will put the address of the first character of `"hello"` into the `%rdi` register and then `callq bar`.
2. `bar` will look in `%rdi` for an integer, modulo it by 10, and use it to put an address of a character in the string `"ten letter"` into `%rax`
3. `baz` will look in `%rax` for an integer, add `x` to it, and return

This is almost certainly not what was wanted,
but no part of it violates the rules.
{/}

<!--
Technically, mis-matched declarations and definitions are undefined behavior,
but unless the compiler has awareness of both files at once
it cannot do anything other than assume all other files agrees with the one it is seeing.
-->

    
## Variadic functions

The number of arguments in a function is known as the functions **arity**.
Many functions have fixed arity, requiring the same number of arguments each time they are invoked,
but sometimes it is nice to have a function that has variable arity, or a **variadic** function.

In C, when invoking a function of variable arity
the invoking code simply follows the calling convention,
putting some arguments in registers and others on the stack.
The invoked function then needs to know how many arguments it received.
Since it can't tell anything without consulting at least one argument,
all variadic functions in C require at least one argument,
and almost all use that argument to decide how many (and what type) the other arguments are.

By far the most famous variadic function in C is `printf`,
which is defined as

````c
int printf(const char *format, ...);
````

Note the trailing `...` means "this is a variadic function."
Thus, `printf` may be invoked with any arguments you want,
as long as the first is a `const char *` (that is, a string):

````c
printf("%s, %s %d, %.2d:%.2d\n", weekday, month, day, hour, min);
````

The `printf` function uses fairly involved rules about `%`s in its first argument
to determine how many and what type the other arguments should be.

Writing a variadic function is somewhat complicated
by the fact that the extra arguments do not have names.
C provides (declared in `stdarg.h`) a special data type `va_list`
and a set of special macros to use in accessing variadic arguments.

````c
void va_start(va_list ap, argN);
type va_arg(va_list ap, type);
void va_end(va_list ap);
````

To use these, you might do something like

````c
int sign_swaps(int num0, ...) {
    va_list ap;
    int last = num0;
    int ans = 0;

    va_start(ap, num0);
    while(last != 0) {
        int next = va_arg(ap, int);
        if ((last < 0) != (next < 0)) ans += 1;
        last = next;
    }
    va_end(ap);

    return ans;
}
````

If you want to write variadic functions, you should

1. Read all of `man stdarg.h` twice
2. Look up variadic security vulnerabilities like the [format string attack](https://en.wikipedia.org/wiki/Format_string_attack)
3. Write good tests, including too-few- and too-many- and wrong-type-argument invocations.

# Preprocessor

Before compilers compile code, they run the C Preprocessor.
This does several uninteresting tasks like removing comments,
but also processes various *macros* and *directives*.

`#include <somefile.h>`
:   Looks for `somefile.h` in the *include path*, a set of directories
    typically including `/usr/include` and sometimes a few others.
    
    Upon finding the file, it dumps its entire contents into this part of the file,
    as if you had copy-pasted it here.

`#include "somefile.h"`
:   Looks for `somefile.h` in the current source directory
    and, if not found there, in the *include path*.
    
    Upon finding the file, it dumps its entire contents into this part of the file,
    as if you had copy-pasted it here.

`#if expression`, `#else`, `#elif expression`, and `#endif`
:   Upon encountering an `#if`, the preprocessor evaluates the truth of the expression,
    which must contain only literals (and operators) because the preprocessor is not running code.
    If it is false, all code from that `#if` to the matching `#else`, `#elif`, or `#endif` is removed from the source code as if you had deleted it.
    
    `#else` and `#elif expression` behave like `else` and `else if (expression)` would in C.

`#define NAME anything at all`
:   Defines an *object-like macro*.
    Anywhere `NAME` appears in the source code, this tells the preprocessor to replace it with `anything at all`---literally those exact tokens,
    as if you had done a global find-and-replace in your source file.
    
`#define NAME(a,b,c) anything including a and b and c`
:   Defines a *function-like macro*.
    Anywhere `NAME(x,y,z)` appears in the source code, this tells the preprocessor to replace it with `anything including x and y and z`---that is,
    it does a find-and-replace with some parameterization.
    
    This is a lexical replacement, not a syntactic one, so you should almost always add parentheses around each argument and around the full expression:
    
    ````c
    #define TIMES2(x)  x * 2        /* bad practice */
    #define TIMES2b(x) ((x) * 2)    /* good practice */
    
    int x = ! TIMES2(2 + 3);   /* int x = ! 2 + 3 * 2;      (i.e., !2 + 6 == 6) */
    int y = ! TIMES2b(2 + 3);  /* int x = ! ((2 + 3) * 2);  (i.e., !8 == 0) */
    ````
    
    If you decide to become a C expert, there is more to know about macros;
    see <https://en.wikipedia.org/wiki/C_preprocessor#Special_macros_and_directives> for a reasonable overview.
    
    
`#ifdef NAME`, `#ifndef NAME`
:   These act like `#if`, except instead of checking if something is true
    they check if a name has been `#defined` (`#ifdef`) or not (`#ifndef`).

    A *very* common use of these macros is to ensure only one copy of an `.h` file is included.
    For example, `my_file.h` might look like
    
    ````c
    #ifndef __MY_FILE_HAS_BEEN_INCLUDED__
    #define __MY_FILE_HAS_BEEN_INCLUDED__
    
    /* file contents here */
    
    #endif
    ````
    
    This way if a file `#include "my_file.h"` twice
    (as, for example, because it includes two other `.h` files that each `#include` `my_file.h`)
    then the first one will define `__MY_FILE_HAS_BEEN_INCLUDED__`
    and the second one, seeing `__MY_FILE_HAS_BEEN_INCLUDED__` is already defined,
    will have all its contents removed by the `#ifndef`.
    
    {.example ...}
    If we have something like
    
    +---------------------+-----------------+
    | foo.c               | foo.h           |
    +=====================+=================+
    |````c                |````c            |
    |int x;               |#ifndef __FOO_H  |
    |#include "foo.h"     |#define __FOO_H  |
    |int y;               |int foo;         |
    |#include "foo.h"     |                 |
    |int x;               |#endif           |
    |````                 |````             |
    +---------------------+-----------------+
    
    the `#include` processing will create

    ````c
    int x;
    #ifndef __FOO_H
    #define __FOO_H
    int foo;
    
    #endif
    int y;
    #ifndef __FOO_H
    #define __FOO_H
    int foo;
    
    #endif
    int x;
    ````
    
    The first `#ifndef` is true, since `__FOO_H` had not been defineed before that line

    ````c
    int x;
    #define __FOO_H
    int foo;
    
    int y;
    #ifndef __FOO_H
    #define __FOO_H
    int foo;
    
    #endif
    int x;
    ````

    That means that the second `#ifndef` is false, since the first defined `__FOO_H`

    ````c
    int x;
    #define __FOO_H
    int foo;
    
    int y;
    int x;
    ````
    {/}
    
    
`__FILE__` and `__LINE__`
:   The preprocessor is guaranteed to define `__FILE__` as an object-like macro expanding to the name of the current file, in quotes,
    like `"my_file.c"`.
    The preprocessor is also guaranteed to define `__LINE__` as an object-like macro expanding to the line number on which __LINE__ appears,
    like `23`.
    
    These are often used in debugging messages, as e.g. `printf("Error in %s on line %d\n", __FILE__, __LINE__);`.
    
    Because the preprocessor redefines these on its own on each new line of code,
    they have a special `#line` directive to change them if you need to do that (not the usual `#define`).
    The `#include` processing and comment removing adds such `#line` directives so that it does not change source line numbers.

`#error "error message"`
:   Shows `error message` as an error message during compilation.

Most C compilers add several other compiler-specific preprocessor directives, like `#warning`, `#pragma message`, `#pragma once`, `#include_next`, `#import`, etc.
Each is added to simplify some common task, but also makes code harder to port to other platforms.
