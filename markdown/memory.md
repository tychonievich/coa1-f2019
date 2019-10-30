---
title: An overview of memory
...

This is intended to be a practical guide (rather than an authoritative guide) to memory allocation and management in C,
as implemented by clang and gcc for the x86-64 processor family under Linux.

# Memory layout

Compiled binaries are typically *relocatable*, meaning they contain only guidelines about where code may be located in memory;
the loader is responsible for assigning specific addresses prior to running the code.

Linux x86-64's loaders provide the following contents of memory, with large addresses at the top:

--------------------------------------------------------------------------------
Address range           Use
-------------------     ------------------------------------------------------------
above 0xFFFFFFFFFFFF    kernel memory (OS runs here; if your code accesses these 
                        segments as e.g. via `*-1`, you code crashes)

below 0xFFFFFFFFFFFF    user stack (grows into smaller addresses)

&nbsp;                  empty space for future stack growth
                                        
&nbsp;                  memory-mapped region (shared libraries)

&nbsp;                  empty space for future heap growth
                        read/write segments (`.data` for initialized globals,
                        `.bss` for uninitialized globals)

&nbsp;                  run-time heap (grows into larger addresses)

above 0x400000          read-only code and data (`.init` gets tarted, `.text` is 
                        your code, `.rodata` is string constants and such)

0x0--0x400000           unused segments, so that `*0` and the like crashes
--------------------------------------------------------------------------------

All of the user-access regions may be randomized (doing so is called ASLR: Address Space Layout Randomization)
to prevent certain families of security vulnerabilities.

# Pointers to `struct`s

It is common to use a pointer to a `struct`.
In fact, it is uncommon to have a `struct` typed variable; almost all `struct`s are handled through a pointer.
But this leads to a syntactic unpleasantness.
Because `.` has higher precedence than `*`, `*a.b` means `*(a.b)`,
so to get a field from a pointer to a `struct` requires parentheses: `(*a).b`.

Because of this, C has a special operator `a->b` that means `(*a).b`.
We use it extensively in place of what Java or Python would do with a `.`.

# Using the stack in C

Although the compiler may optimize this by placing variables in registers,
conceptually all local variables (including parameters and temporary values created as part of a computation)
are stored on the stack.
Because you've already learned to write programs with local variables, you've also already learned to use stack memory.

Stack memory is automatically "allocated" when a function is invoked and "de-allocated" when a function returns,
although this does not actually entail much work beyond changing the contents of `%rsp`.
Because of this, you should **never return the address of a local variable**.

{.example ...} Consider the following program:

````c
int *makeArray() {
    int answer[5];
    return answer;
}
void setTo(int *array, int length, int value) {
    for(int i=0; i<length; i+=1) array[i] = value;
}
int main(int argc, const char *argv[]) {
    int *a1 = makeArray();
    setTo(a1, 5, -2);
    return 0;
}
````

The function `makeArray` will allocate room for 5 `int`s on the stack (e.g., by adding `-20` to `%rsp`)
and return the address of those `int`s.
The function `setTo` will then be invoked, with its first argument being a pointer to *it's own stack frame*
as `setTo` reuses the same stack memory that `makeArray` used.
If `setTo` stores `i` on the stack (as opposed to optimizing it into a register), `setTo` will not work properly.
For example, we might see something like

Address     makeArray's use             setTo's use
----------  --------------------------  -----------------------------------------------
...200      return address              return address
...1F8      saved copy of `%rbp`        saved copy of `%rbp`
...1F4      allocated as `answer[4]`    pointed to by `array[4]` *and* allocated as `i`
...1F0      allocated as `answer[3]`    pointed to by `array[3]`
...1EC      allocated as `answer[2]`    pointed to by `array[2]`
...1E8      allocated as `answer[1]`    pointed to by `array[1]`
...1E4      allocated as `answer[0]`    pointed to by `array[0]`

... which would mean that in `setTo` the values of `i` will repeat an infintite loop:
0, 1, 2, 3, 4, in the usual way, but then iteration `i=4` will assign to `array[4]` which is address ...1F4 which is also the value of `i`, setting it to -2 and causing the loop to repeat as -1, 0, 1, 2, 3, 4, -1, 0, 1, ... forever.

Note that this bug may become invisible if we compile with optimizations and `i` is stored only in a register;
we still did the wrong thing in `makeArray`, so this is still a bug, but we might not see it in this program.
{/}

# Using global variables in C

Global variables in C are allocated in regions of memory that are accessible to all functions,
which memory is set aside when the program is compiled and thus must be of a size the compiler can determine at compile time.
Use of global variables can be an efficient way to program,
provided you know in advance how much memory you'll need.

A common pattern in using global arrays is to (a) `#define` a maximum size and (b) use a variable to track how much has actually been used.

{.example ...} The following is a simplified partial example of how one might collect a set of courses a student is interested in:

````c
/* A function we'd need to define elsewhere that reads up to max chars *
 * from the keyboard into an array, returning how many were read       */
unsigned get_input(char *dest, unsigned max);

#define MAX_CLASS_SIZE 12
#define MAX_CLASSES    20

/* Note: 2D arrays are declared with sizes in the same order as indices */
char interest[MAX_CLASSES][MAX_CLASS_SIZE];
unsigned classes = 0;

int main(int argc, const char *argv[]) {
    while (classes < MAX_CLASSES) {
        puts("What class are you interested in? Press enter when done.");
        unsigned got = get_input(interest[classes], MAX_CLASS_SIZE);
        if (got == 0) break;
        else classes += 1;
    }
    
    puts("You expressed interest in the following:");
    for(int i=0; i<classes; i+=1) {
        puts(interest[i]);
    }

    return 0;
}
````

Correct implementation of functions like `get_input` will be a subject for a future part of this course.
{/}

Because global arrays are simple to program and efficient in practice,
they are common in C code.
Because a global array typically needs associated global information like used size,
that means other global variables are also common.

A sizable (though not unanimous) majority of software engineering text I have consulted
explicitly state that "global variables are **bad**."
One of the more readable examples I've found is <http://wiki.c2.com/?GlobalVariablesAreBad>.
However, even when well written these tend to refer to topics like "coupling" and "namespace polution"
that are difficult to motivate properly before you've work on large software projects yourself.

# Using the heap in C

If the stack cannot be used for long-term pointers
and global variables have to be allocated at compile time and may also be bad for software maintenance,
how do you allocate memory as the program runs and pass it around to different functions?
The answer: put it in/on^[When a value is stored in memory that is part of the heap,
it roughly equally common to refer to the value as being "on the heap" or "in the heap".
There appears to be a slight preference for "on" to refer to the memory itself
and "in" to refer to the values stored using that memory, but many exceptions exist.] the heap.

The heap is a region of memory where

- any number of chunks of memory of any size and purpose may co-exist
- new chunks can be added as the program runs
- each chunk remains until it is explicitly deallocated or the program terminates

{.aside ...}
"Heap" is used in two main ways in programing.
When discussing memory, "the heap" is an unorganized region of memory made out of many heterogeneous chunks of memory with different purposes.
When discussing data structures, "a heap" is a partially-organized tree structure where "small" things make their way to the top without requiring complete ordering of the whole.
There is no relationship between these: *the* heap is not *a* heap, and *a* heap need not be stored in *the* heap.
This course will only use it in the former way: a region of memory, not a data structure.
{/}

## Managing memory

The operating system has final say on what memory, and how much of it, each program gets.
It handles this via several concepts, including *virtual addresses* that ensure that two different processes cannot access one another's memory and *segments* that prevent you from jumping to an array or dereferencing a pointer to unallocated memory.

Operating systems typically allocate memory in large regions called "pages".
It is common today^[a.d. 2018] for pages to be 4KB -- that is, 4096 bytes.
Programs can ask the operating system for more pages of memory, or return pages of memory to the OS.
However, programmers typically want to handle memory at finer-grained resolution,
and C provides library functions to assist with this.

### `malloc` (memory allocate)

The library function `void *malloc(size_t size);` returns a pointer to the first byte of a `size`-byte region of memory
that is allocated on the heap and not used by any previous purpose.
It does this by

- Checking to see if it has enough space on partially-used heap page. If no, ask the OS to allocate new pages until it has enough unused heap space.
- Pick an address to return.
- Add that address and its allocated size in a special bookkeeping data structure.
- Return the address.

The internal bookkeeping data structure allows subsequent calls to `malloc` to be guaranteed not to return the same (or an overlapping) region a second time.

{.example ...} It is typical to `malloc` a `struct` with `sizeof` and a pointer type cast, like

````c
typedef struct student_s {
    const char *name;
    int credits;
} student;

student *enroll(const char *name, int transfer_credits) {
    student *ans = (student *)malloc(sizeof(student));
    ans->name = name;
    ans->credits = transfer_credits;
    return ans;
}
````
{/}

{.example ...} It is typical to `malloc` an array with `sizeof` and a multiplier, like

````c
typedef struct length_array_s {
    long *data;
    unsigned capacity;
    unsigned size;
} larray;

larray *make_array() {
    larray *ans = (larray *)malloc(sizeof(larray));
    ans->size = 0;
    ans->capacity = 16;
    ans->data = (long *)malloc(sizeof(long) * ans->capacity);
    return ans;
}
````
{/}


With very rare exceptions, we malloc to store one or more values of a given size inside the malloced memory; if you find yourself `malloc`ing *without* a `sizeof` inside, you almost certainly did something wrong.

### `free`

The library function `void free(void *);` accepts a pointer returned by `malloc` and marks it as no longer in use, and hence as available for future `malloc`s.

In small, short-running programs you may be able to get away with never `free`ing your data structures,
but in larger and longer-running programs this can cause a program to hog all available memory on the computer, slowing all operations and possibly even crashing the program or entire computer.
Programs that allocate memory and then forget about it without `free`ing it are said to have a [Memory leak]

### `calloc` and `realloc`

Two additional convenience functions can also be useful.

`x = calloc(n, s);` is the same as `x = malloc(n * s);` except that it (a) may be optimize for storing an array of `n` distinct `s`-byte values and (b) sets all bytes of allocated memory to `0`.
Notably, `malloc` does *not* erase the memory it returns.

{.example ...}
After running the following code:

````
int *x = (int *)malloc(sizeof(int));
int *y = (int *)calloc(1, sizeof(int));
int a = *x;
int b = *y;
````

the value of `a` may be anything, while `b` is guaranteed to be `0`.
{/}


`x = realloc(x, s);`

- *either* extends the previously-allocated region pointed to by `x` to be `s` bytes long,
- *or* 
    1. allocates a new `s`-byte region,
    2. copies the bytes previously pointed to by `x` into this new region, 
    3. `free(x)`, and then 
    4. returns the new region's address.

{.example ...}
After running the following code:

````
int *x = (int *)calloc(8, sizeof(int));
x[4] = 123;
int *x = (int *)realloc(x, 16*sizeof(int));
````

`x` is a pointer to an array of 16 `int`s.
The first 8 elements are `{0, 0, 0, 0, 123, 0, 0, 0}`
and the next 8 could be anything.
{/}

## Garbage collectors

Many languages do not have an equivalent to `free`.
They let you allocate memory, but never ask you to deallocate it.
They avoid (most) memory leaks by adding to your program a *garbage collector*.

### Garbage

Memory is **garbage** if it is (a) allocated on the heap and (b) will never be used by your program again.

Memory is **unreachable** if it is (a) allocated on the heap and (b) it is not part of a *reachable* allocated memory block.
A block of memory returned by a single call to `malloc` or its friends is **reachable** if any of the following are true:

- its address is in a program register, or
- its address is on the stack, or
- its address is in a reachable block of memory

All *unreachable* memory is *garbage*, but not all *garbage* is *unreachable*.

{.example ...}
Consider the following code:

```c
int bad(int a) {
    int *list = malloc(a, sizeof(int));
    for(int i=0; i<a; i+=1) list[i] = (i+1*(i+1);
    int sum = 0;
    for(int i=0; i<a; i+=1) sum += list[i];
    // midpoint
    int ans = 0;
    while (sum > 0) {
        ans += 1;
        ans >>= 1;
    }
    return ans;
}
```

The memory returned by `malloc` becomes garbage at the comment `// midpoint`{.c} because it is never used after that;
it becomes unreachable after the function returns (and hence is a [memory leak](#memory-leak)).
{/}

### Garbage detection

There are several well-known, well-studied, and carefully-implemented algorithms for performing garbage detection; almost all of these detect only unreachable garbage.
A garbage **garbage collector** is a process that detects garbage and then frees it; the most common model (technically a "tracing garbage collector") works as follows:

- inspect the entire contents of a program's memory
- flag as garbage all unreachable memory on the heap
- `free` that garbage

Thee steps require significant bookkeeping data structures and processing power,
and (b) periodically pausing the entire program to perform a garbage detection hunt^[There are garbage collectors that run *while* the rest of the program is modifying memory, but doing so has various challenges that make them more complicated and have various possible drawbacks. See <https://en.wikipedia.org/wiki/Tracing_garbage_collection#Stop-the-world_vs._incremental_vs._concurrent> for more.].
In general, this can slow down a program, and increase the memory is uses,
and cause it to pause at awkward times.
As garbage collectors become more sophisticated and computer memory becomes cheaper,
these concerns are decreasingly important;
and the ability to write code that does not need to worry about `free`ing unused memory
is a definite plus for software developers.
However, because garbage collection always requires some space and time overhead,
and because every byte and cycle always matters for some programmers somewhere,
languages like C that do not have garbage collection remain common.


# Detecting and avoiding bugs

This section is devoted to common mistakes people make when handling memory.
Some of it is a duplication of material above, re-phrased here for inclusion in the general category of "bugs".

## Using the "address sanitizer"

Most current compilers come with an option to compile in to the binary
some additional information that can detect many common kinds of memory errors.

If you compile using `clang`, you can add `-fsanitize=address` to add in these checks.
To get useful error messages when a problem is found with your code,
you should also add `-g` and `-fno-omit-frame-pointer`

Note that the address sanitizer inserts bug detection code into the binary at compile time,
but only actually detects bugs when the compiled program is run.
Because of this, bugs that exist but that your program doesn't use (e.g., because they are in a branch of an `if` statement that your test cases do not exercise) are not detected.

{.aside ...}
Both `gcc` and `clang` have a variety of different categories of command-line flags.
Often the first letter tells you something about the flag:

- `-f...` enables or disables a specific feature, such as an optimization, protection, or syntax extension.
- `-m...` changes some aspects of the ISA code is generated toward
- `-O...` selects a group of commonly-used optimizations (some of which are individually selectable using `-f...` options)
- `-W...` controls what warning messages are displayed
- `-g...` specifies how much debugging information should be generated and included in the binary

{/}

## Common kinds of problems

The following are brief descriptions of several common memory bugs.


### Memory leak

A memory leak occurs when you fail to `free` [garbage](#garbage-collectors)
or otherwise keep un-`free`d heap allocations of un-used memory.

The [address sanitizer](#using-the-address-sanitizer) can detect this bug,
but is somewhat conservative in what it looks for.
Often it is necessary to explicitly change a pointer before the sanitizer notices the leak.

{.example ...}
````c
/** represents a mathematical expression */
typedef struct expr_s {
    char kind; // '=' for literal, or '+', '-', or '*' for operators
    long value;           // only used by '='
    struct expr_s *left;  // only used by operators
    struct expr_s *right; // only used by operators
} expr;

/** turns an expression of literals into a literal */
long flatten(expr *e) {
    if (e->kind == '+') {
        e->kind = '=';
        e->value = flatten(e->left) + flatten(e->right);
        
        /* memory leak: remove pointers without free 
         * asan will only notice it because of the explicit = NULL */
        e->left = e->right = NULL;
    } else if (e->kind == '-') {
        e->kind = '=';
        e->value = flatten(e->left) - flatten(e->right);\
        free(e->left);
        free(e->right);
        e->left = e->right = NULL;
    } else if (e->kind == '*') {
        e->kind = '=';
        e->value = flatten(e->left) * flatten(e->right);
        free(e->left);
        free(e->right);
        e->left = e->right = NULL;
    }
    return e->value;
}
````
{/}

Memory leaks tend to make the program use more and more memory,
becoming slower and slower the longer it runs.
In the worst case, this can even cause your entire system to grind to a halt.

[Garbage collectors] are often said to remove the chance of memory leaks,
but this not not strictly true: they measure *reachability* of heap memory, not *possibility of future use*.
Even when writing in Java, Python, or other garbage-collected languages,
make sure you set unused references to objects to `None`/`null`
and otherwise don't maintain references to data you will not reuse.

### Uninitialized memory

Because `malloc` and `realloc` do not initialize the memory they allocate,
it is an error to access that memory before you initialize it.
This is also true of local or global variables, structs, and arrays.
Using uninitialized memory is a particularly tricky bug to notice
because it is often the case that for many runs in a row the uninitialized memory
just happens to be all `0` bytes,
and then one time it happens to be other values instead, causing the bug to manifest itself intermittently.

{.example ...}
````c
int *allsum(int *y, int n) {
    int *x = (int *)malloc(sizeof(int)*n);
    for (int i=1; i<n; i+=1)
        for (int j=0; j<i; j+=1)
            x[j] += y[i];
    return x;
}
````
{/}

### Accidental cast-to-pointer

If a function expects a pointer and you give it an integer instead,
it will interpret the integer value as being an address.
This is particularly problematic with variadic functions like `printf` and `scanf`
that are harder for the compiler to type-check.

{.example ...}
````c
int x; scanf("%d", x); // should have been scanf("%d", &x);

int x; printf("%s", x); // %s means "char *" not "int"
````
{/}

### Wrong use of `sizeof`

It is fairly common to make mistakes with `sizeof`, such as

-   using `sizeof(T)` when you meant `sizeof(T *)`

    {.example ...}<span class="gap"> </span> `int **A = (int **)malloc(sizeof(int) * n);`{.c}
    {/}

-   using `sizeof(T)` when adding to a `T *`

    {.example ...}
    ````c
    int *find(int *p, int val) { 
        while(*p && *p != val) p += sizeof(int);
        return p;
    }
    ````
    {/}

-   failing to use `sizeof` when `malloc`ing

    {.example ...}<span class="gap"> </span> `int *ten_ints = (int *)malloc(10);`{.c}
    {/}


### Unary operator precedence mistakes

Most programmers have a hard time remembering the order of operations between prefix and postfix unary operators.
Is `&a.b` `(&a).b` or `&(a.b)`?
Is `*a++` `(*a)++` or `*(a++)`?
Etc.

This lack of clarity leads to programmer mistakes that can cause many kinds of problems;
when it includes modifying (or failing to modify) a pointer, those problems can become memory errors.

A few suggestions to avoid these:

- If you have prefix- and postfix-operators, always include parentheses to keep them separate
- Avoid postfix `--` and `++` unless you actually need their return-previous-value semantics
- Make use of `->` instead of a combination of `*` and `.` whenever you can


### Use after free

After you `free` a block of memory, using a pointer to it is an error.

The [address sanitizer](#using-the-address-sanitizer) is usually able to detect this bug.

{.example ...} The following is a minimal example
````c
int *x = (int *)malloc(sizeof(int)*10);
int *y = &(x[5]);
free(x);
int z = *y;
````

More realistic examples generally hide the copying of the pointer and the freeing of its target memory
inside other custom functions.
{/}


### Stack buffer overflow

If you index past the end of a stack-allocated memory region, this is called a "stack buffer overflow".
This rarely crashes a program itself, but usually messes up what it will do in the future
by changing the value of some other local variable or overwriting the return address.

Changing the return address usually causes a segfault when you `retq`,
but stack buffer overflow can also allow malicious users to take over your program
by intentionally supplying a return address that causes `retq` to jump to an address of code they included in the buffer overflow or some other code you didn't want to run.

The [address sanitizer](#using-the-address-sanitizer) is usually able to detect this bug.

{.example ...}
````c
char word[16];
scanf("%s", word); // overflows if type a 16+-character word
````

Since `scanf`'s `%s` format specifier reads a non-whitespace sequence of characters into `word`,
this will be a buffer overflow if you type sixteen or more characters without any whitespace.
{/}

### Heap buffer overflow

If you index past the end of a heap-allocated memory region, this is called a "heap buffer overflow".
This can "corrupt the heap" -- that is, the program continues to run,
but the overflow modified some other data structure,
messing up some other part of your program.

The [address sanitizer](#using-the-addresssanitizer) is usually able to detect this bug.

{.example ...}
````c
char word = (char *)malloc(16 * sizeof(char));
scanf("%s", word); // overflows if type a 16+-character word
````
{/}


### Global buffer overflow

If you index past the end of global memory region, this is called a "global buffer overflow".
This sometimes causes a segfault, or it might overwrite a different global variable.

The [address sanitizer](#using-the-address-sanitizer) is usually able to detect this bug.

{.example ...}
````c
char word[16];
int f() {
    scanf("%s", word); // overflows if type a 16+-character word
}
````
{/}


### Use after return

If you return the address of a local variable, and then later use that pointer,
you have a use-after-free bug.

The [address sanitizer](#using-the-address-sanitizer) is usually able to detect this bug.

See the worked example in the section [Using the stack in C] above.


### Uninitialized pointer

If you dereference a pointer that you failed to initialize,
you are likely to end up in an invalid code segment and get a segfault;
however, you might by random bad luck end up with a pre-initialized value that points to valid memory
and end up overwriting a value some other part of the program depends on.

{.example ...}
````c
int *x; int y = *x;    // a fairly obvious bug...

printf("%s");          // printf's %s means "a char *" which we failed to supply
````
{/}

### Use after scope

This is a nuance related to use-after-free.

Each set of braces and each for loop creates its own variable scope.
The compiler is free to re-use that stack space after the scope ends if it wants.
If you use a pointer to an out-of-scope variable, this creates a user-after-scope bug.

The [address sanitizer](#using-the-address-sanitizer) is able to detect this bug,
but requires a special additional flag during compilation to do so: `-fsanitize-address-use-after-scope`.

{.example ...} The following code may or may not have this bug, depending on how the compiler choses to optimize it.
````c
int *p;
{
    int x = 0;
    p = &x;
}
*p = 5;
````
{/}


