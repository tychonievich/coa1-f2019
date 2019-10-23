---
title: Small C functions
...

Write a C file named `smallfunc.c` that contains implementations of **at least five** of the following eight functions.

When you compile, make sure to use the `-m64` flag to ensure that `long` is a 64-bit number and `void *` a 64-bit pointer.

No function you write in this file should invoke a function you did not write.

When you submit **do not include main** in your file. You are welcome to add it for testing, but remove it before submitting.

> The submission server is configured to test your code and give preliminary feedback within 15 minutes of recieving a submission. We encourage you to submit each time you think you've completed one of the functions so that you can verify that it is working according to our tests.

# `void capitalize(char *s);`

Write a function with the above signature
that converts all lower-case letters in `s` into upper-case letters.

{.aside ...}
You did this by hand [Lab02](lab02-hex-editor.html).
{/}

{.example ...}
The following usage code

````c
char *s = strdup("the book \"The C Programming Language.\"");
printf("before: %s\n", s);
capitalize(s);
printf("after:  %s\n", s);
free(s);
````

will display 

    the book "The C Programming Language."
    THE BOOK "THE C PROGRAMMING LANGUAGE."
{/}

Tip: test this with empty strings, strings with digits and punctuation in them, etc.


# `void fibarray(unsigned char *dest, unsigned num);`

Write a function with the above signature
that places the first `num` fibonacci numbers (modulo 256, since the array stores only bytes) into `dest`.

{.aside ...}
You implemented something very like this in [PA04](pa04-fib.html). Unlike PA04, however, your code must work for any location and number of values requested.
{/}

{.example ...}
The following usage code

````c
unsigned char a[64];
fibarray(a, 64);
for (int row=0; row<4; row+=1) {
    for (int col=0; col<16; col+=1) {
        printf(" %02hhx", a[row*16 + col]);
    }
    printf("\n");
}
````

will display

     01 01 02 03 05 08 0d 15 22 37 59 90 e9 79 62 db
     3d 18 55 6d c2 2f f1 20 11 31 42 73 b5 28 dd 05
     e2 e7 c9 b0 79 29 a2 cb 6d 38 a5 dd 82 5f e1 40
     21 61 82 e3 65 48 ad f5 a2 97 39 d0 09 d9 e2 bb
{/}

Tip: test this with 0- and 1-entry arrays as well as larger ones, and make sure you don't set more values of the array than the requested `num`.

# `long recpow(long x, unsigned char e);`

Write a function with the above signature
that computes and returns `x`^`e`^.
You **must** implement this recursively, not iteratively (do not use `for`, `while`, `do`, or `goto`).

{.aside ...}
You implemented something very like this in [PA05](pa05-assembly.html). Unlike PA05, however, your code must work for any `x` (including negative and zero) and any `e` (including zero).
{/}

{.example ...}
The following usage code

````c
long x = -21;
unsigned char e = 13;
long ans = recpow(x, e);
printf("%ld\n", ans);
printf("%ld\n", recpow(11, 0));
````

will display 

    -154472377739119461
    1
{/}

Tip: test with both negative and positive `x` and both even and odd `e`.

# `unsigned long nextprime(unsigned long x);`

Write a function with the above signature
that returns the smallest prime number larger than `x`.

{.example ...}
The following usage code

````c
long x = 100;
for (int i=0; i<10; i+=1) {
    x = nextprime(x);
    printf("%ld\n", x);
}
printf("%ld\n", nextprime(1000000000000));
````

will display 

    101
    103
    107
    109
    113
    127
    131
    137
    139
    149
    1000000000039

Note: if your code is not very efficient, the last number might not appear for so long you lose patience waiting for it. That's OK; we'll only test with inputs that end in a reasonable time.
{/}

Tip: A number *x* is prime if and only if it is 2, or it is odd and is not a multiple of any odd number between 3 and √*x*.

Tip: Make sure you test with the arguments `0`, `1`, and `2`.

# `unsigned long binterleave(unsigned x, unsigned y);`

Write a function with the above signature
that returns a 64-bit number created by interleaving the bits of the two arguments.
The low-order bit of the result should be the low-order bit of `x`;
then the low-order bit of `y`,
then the next-to-lowest bit of `x`,
then the next-to-lowest bit of `y`,
and so on up to high-order bit of `y`.

You should use at least one loop in your solution.

{.example ...}
The following usage code

````c
printf("%lx\n", binterleave(0x100030f, 0x1003f0));
printf("%lx\n", binterleave(0xffffff0f, 0x00000000));
printf("%lx\n", binterleave(0x00000000, 0xffffff0f));
````

will display 

    10200000faa55
    5555555555550055
    aaaaaaaaaaaa00aa
{/}

Tip: Remember that most operations, if given only 32-bit arguments, will truncate their result to 32 bits. You'll probably want to make sure you have a `long` value (such as `0L`) involved in important operations in your code.

Tip: You almost certainly want to test this with smaller numbers than the example uses first, so that you can more easily convert them to binary to check your work.


# `void reverse(int *arr, unsigned length);`

Reverse the fist `length` elements of `arr` in place.

{.example ...}
The following usage code

````c
int x[] = {1, 1, 2, 3, 5, 8, 13, 21};
for (int i=0; i<8; i+=1) printf("%d, ", x[i]); printf("\n");
reverse(x, 6);
for (int i=0; i<8; i+=1) printf("%d, ", x[i]); printf("\n");
````

will display 

    1, 1, 2, 3, 5, 8, 13, 21, 
    8, 5, 3, 2, 1, 1, 13, 21, 
{/}

Tip: Test this with both even and odd array lengths.

# `void push0(int *arr, unsigned length);`

Rearrange the first `length` values of `arr` in place, such that all of its non-zero values
appear in their original order, followed by all of its zero values.

{.example ...}
The following usage code

````c
int x[] = {1, 7, 3, 2, 0, 5, 0, 8, 0, 7, 5, 6, 8, 8, 7, 7, 2, 9};
for (int i=0; i<18; i+=1) printf("%d ", x[i]); printf("\n");
push(x, 15);
for (int i=0; i<18; i+=1) printf("%d ", x[i]); printf("\n");
````

will display 

    1 7 3 2 0 5 0 8 0 7 5 6 8 8 7 7 2 9
    1 7 3 2 5 8 7 5 6 8 8 7 0 0 0 7 2 9
{/}

Tip: Test this with both arrays that do and don't include zeros.

# `int nondup(int *arr, unsigned length);`

Assume that `arr`'s first `length` elements contains exactly two copies of each value it contains, except one value it has only once.
Return that one non-duplicated element.

If the array does not have a unique non-duplicated element, the behavior of `nondup` is undefined.

{.example ...}
The following usage code

````c
int x[] = {28, 12, 8, 0, 0, 28, 8};
printf("%d\n", nondup(x, 7));
printf("%d\n", nondup(x + 2, 5));
````

will display 

    12
    28
{/}


Tip: There are (at least) two very different solution approaches: one with nested loops and another using xor.
