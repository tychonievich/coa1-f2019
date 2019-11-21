---
title: From C to C++
...

The goal of this lab is to explore how C++ adds minimal extensions to C in order to provide several features that have become common in object-oriented languages.
The subsequent lab will have you use these features to work with some basic C++ code.

# How this lab works

## Passing off

Each subsequent section of this lab writeup describes a C++ feature and ends with a set of questions about that feature.
You'll pass off the lab by sharing your answers to a TA-selected subset of these questions with a TA.
The TA may also ask a variation on one or more question to ensure understanding of the underlying concepts.

We recommend you write down your answers as you determine them.

## Compiling

Many parts of this writeup will ask you to inspect various aspects of compiled C++ code.
You can run a C++ compiler by using `clang++` (or `g++`);
to access the resulting assembly use the `-S` flag (e.g., `clang++ -S ex.cpp`{.bash} generates assembly in `ex.s`);
to inspect the data set by the code use `lldb` (or `gdb`).

You'll likely find that `-O1` will generally create much shorter code without removing useless code that you might have wanted to test something.

You should always give your C++ files the extension `.cpp`.
Some 

# Function name overloading

In C, if you name a function `baz` then it is compiled with label `baz`.
That means you cannot re-use the name `baz` for more than one function.

In C++, function names are only part of the compiled label:
argument types and where the name appeared are also part of it.
This allows code like

```cpp
int abs_val(int x) { return x < 0 ? -x : x; }
double abs_val(double x) { return x < 0 ? -x : x; }
int main() {
    printf("%d %f", abs_val(-23), abs_val(-20.3f));
}
```

to be compiled; the compiler picks which version of `abs_val` to invoked in each instance using information derived during type checking.

This renaming of functions to arrive at labels is called **name mangling**.

{.exercise ...}Experiment with the compiler until you are ready to answer the following questions

1. What is the label used by `void f()`{.cpp}?

1. If you use the label name you found in the previous question as a C++ function name, what label is that given?

1. What are the labels given to the following?
    
    C++ function header                         Label
    ------------------------------------------  ----------------------------
    `void f()`{.cpp}
    `void fame()`{.cpp}
    `int f()`{.cpp}
    `float f()`{.cpp}
    `double f()`{.cpp}
    `void f(int a)`{.cpp}
    `void f(int b)`{.cpp}
    `void f(float a)`{.cpp}
    `void f(double a)`{.cpp}
    `void f(int a, int b)`{.cpp}
    `void f(int a, float b)`{.cpp}
    `void f(float a, int b)`{.cpp}
    `void f(float a, int b, double c)`{.cpp}

1. If the TA writes an arbitrary signature using `int`, `float`, and `double` in arguments and return types, could you write its label?
{/} 

# Namespaces

In C, every function and global variable's name must be unique among all the files you link with.
When you write all your own code, that's not too hard to do; but when you are using third-party libraries there is no reason to expect them not to conflict in names.
Because of this, major C libraries generally pick some kind of prefix to all function names,
but it can be annoying to read and write code when virtually every function starts with the same prefix.

Later languages have added in a full module or package system, with full scoping and so on,
but C++ uses a simpler mechanism called **namespaces**.
These are included in the name mangling to further decrease the chance of name collisions.

Name mangling for namespaces uses a length-before-name technique, so that namespace `welcome` would become `7welcome` in the mangled name. This is important to be able to have numbers in the namespace hierarchy: `7welcome2it` represents the two nested namespaces `welcome::it` while `10welcome2it` is the single namespace `welcome2it`

{.exercise ...}Experiment with the compiler until you are ready to answer the following questions

1. What are the mangled names of both `baz`s and `xyxxy` in the following code?
    
    ````cpp
    namespace bar {
        int baz(int x) { return x + 2; }
        namespace flub {
            int baz(int x) { return x - 2; }
        }
        int xyxxy(int, int);
    }
    int bar::xyxxy(int a, int b) { return bar::baz(a) * bar::flub::baz(b); }
    ````

1. Compile and inspect the assembly created by the following. What does the `using namespace` keyword appear to do?

    ````cpp
    namespace a { 
        int b(int c) { return -c * (c-2); }
    }
    int d(int e) { return e+e*e; }

    using namespace a;

    int f(int g) { return d(b(g)); }
    ````
{/}

# Operator overloading

C++ allows you to redefine what almost every operator does.
You can define functions whose name is `operator+`, `operator<<=`, etc,
and they will be used in place of built-in behavior for `+`, `<<=`, etc, when those operators are applied to the appropriate types.

{.exercise ...} Consider the following code:

```cpp
#include <stdio.h>

typedef struct nonint { int x; } nonint;

nonint operator+ (nonint a, nonint b) {
    nonint ans = { a.x - b.x };
    return ans;
}

int main() {
    nonint a = { 3 };
    nonint b = { 5 };
    nonint c = a + b;
    printf("%d + %d = %d\n", a.x, b.x, c.x);
}
```

1. What does it print when run?
1. What assembly instruction does the `a + b` compile to?
{/}

# Other odds and ends

C++ compilation also makes use of a few other syntactic sugar pieces to help make code easier to write.
One example is the "pass by reference" syntax, a short-hand way of using pointers.
For example, what in C might be

```c
char *strsep_c(char **txt, char delim) {
    if (!txt || !*txt) return 0;
    char *ans = *txt;
    while(**txt && **txt != delim) *txt += 1;
    if (**txt == delim) {
        *txt = 0;
        *txt += 1;
    }
    return ans;
}
```

can be written in C++ as

```cpp
char *strsep_c(char *& txt, char delim) {
    if (!txt) return 0;
    char *ans = txt;
    while(*txt && *txt != delim) txt += 1;
    if (*txt == delim) {
        txt = 0;
        txt += 1;
    }
    return ans;
}
```

These two compile to virtually the assembly except that the compiler takes care of ensuring that the second, which uses pass-by-reference syntax, is always given a valid reference so the two-part check in the first is just a 1-part check in the second

Recent versions of C++ have added many other similar tricks for efficient coding with more compile-time checking,
such as more careful casts, self-deallocating pointers, etc.

{.exercise ...}
Write a pass-by-reference wrapper around `realloc` named `realloc2`
such that you can replace `x = realloc(x, 20)` with just `realloc2(x, 20)` without errors.
You'll show the TA your source code.
{/}

# Classes

C++ is often characterized as "C supporting objected oriented coding".
This section explores several of those concepts:

C++ adds a `class` keyword that is identical to `struct` in almost every way,
except a `class` defaults to `private:` members and a `struct` to `public:`.
C++ also removes the need to `typedef` to get a simple name for a `struct` or `class`.

## Member functions

C++ also extends `struct` and `class` to allow them to declare member functions as well as member attributes:

+-----------------------+-------------------------------+
| `.hpp` header         | `.cpp` implementation         |
+=======================+===============================+
|````cpp                |````cpp                        |
|class gleam {          |int gleam::inOrder() {         |
|    int a;             |    return this->a <= this->b; |
|    int b;             |}                              |
|public:                |````                           |
|    int inOrder();     |                               |
|};                     |                               |
|````                   |                               |
+-----------------------+-------------------------------+

{.exercise ...} Compile the above code and inspect the resulting assembly to answer the following:

1. How many arguments does `inOrder` use?
    Recall the arguments are passed (in order) in `%rdi`, `%rsi`, `%rdx`, `%rcx`, `%r8`, %`r9`, and the stack.
    You can determine how many are used by checking how many of these the code for `inOrder`
    accesses before they are set inside that code.

1. From the code that uses each argument, what is the type of each (address, int, etc)?
{/}



## constructors, destructors, `new` and `delete`

Constructors are an important part of how OO objects are initialized.
C++ has added an operator `new` that acts like "`malloc`^[C++ generally manages the heap slightly differently when using `malloc` vs `new`, so never `free` something allocated with `new` or vice verse.] and then invoke a constructor"
and an operator `delete` that acts like "`free` and then invoke a destructor",

Thus, we could test the code above with

```cpp
gleam *a = new gleam();
int b = a->inOrder();
```

To add a constructor, make a member function with the same name as the class; a destructor's name is preceded by a tilde `~`.
Neither should have a return type specified.
Destructors are usually used to `delete` anything the constructor `new`ed,
and are typically omitted if there is no need to do that.

```cpp
class gleam2 {     
    int a;        
    int b;        
public:           
    int inOrder();
    gleam2(int x, int y) { this->a = x; this->b = y; }
    ~gleam2() { puts("Gone"); }
};
```

{.exercise ...}
Given the above code, what assembly operation(s) is/are invoked if we invoke `gleam2 *bazzle = new gleam2(1, 2)`{.cpp}?
{/}

## Inheritance and overloading

We'll talk about how C++ implements inheritance and overloading in lecture.

## Templates

C++ also has a "template language",
which looks superficially like Java's generic syntax but is actually a full programming language in its own right.
Most casual C++ programmers I know only use the basic type-generic aspect of templates,
not any of their advanced features.
