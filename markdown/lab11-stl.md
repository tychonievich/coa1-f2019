---
title: The C++ STL
...

The goal of this lab is to provide enough exposure to C++ that you understand how to explore more on your own.
It assumes you learned all of the topics in [last week's lab](lab10-cpp.html).

# Three Language Features

## Reference arguments

In C, if you have an argument of type `int *`{.c} you don't know if it is an array or just a value you are supposed to set.
C++ adds syntax to allow these to look different the "reference" argument type.

If you write a function where an argument name is followed by an ampersand (e.g., `int x&`{.cpp})
then C++ will compile it to be a pointer, but have your syntax look like it is a variable.

{.example ...} The following two pieces of code are equivalent in how they run

+---------------------------------------+---------------------------------------+
| C code                                | C++ code                              |
+=======================================+=======================================+
| ````c                                 | ````cpp                               |
| int f(int *x) {                       | int f(int x&) {                       |
|     int ans = *x + 2;                 |     int ans = x + 2                   |
|     *x = 3;                           |     x = 3;                            |
|     return ans;                       |     return ans;                       |
| }                                     | }                                     |
| ````                                  | ````                                  |
+---------------------------------------+---------------------------------------+

Both pieces of code turn into equivalent *assembly*, but the C++ version uses *type-checking* to ensure that `x` is never treated as if it were an array instead of a single reference.
{/}

## Templates

As discussed last week, C++ allows function names to be overloaded using name mangling.
It also allows more complicated overloads.

A **template** in C++ is an outline of how to create code if it is needed.
Templates uses angle-brackets, like generics in Java.
Java, however, discards the generics during compilation as part of the type checking step.
C++ instead uses them to generate special code for each template argument created.
Thus, C++ code like

```cpp
template <typename T> T foo(T x) { /* ... */ }
```

does not create any code, by itself.
Instead, each time you use the template with different template parameters
the compiler creates code for that parameterization.
Thus, if you later have 

```cpp
int x = foo(3);
double y = foo(3.14);
```

the compiler will create two different `foo` functions:
one that has an `int` parameter and return type
and another that has `double`.

Note that the compiler will try to guess what template parameters were missing; if you want to be explicit, you can also write

```cpp
int x0 = foo(3); // compiler figures out it is a template, and deduces type
int x1 = foo<>(3); // told it is a template, compiler deduces type
int x2 = foo<int>(3); // told it is a template and type
```

{.aside} Another difference from Java is that the parameters of a template do not need to be type names; they could also be values, like integers. Value templates are unusual enough in C++ we'll say no more about them here.

{.aside ...} C++ template metaprogramming is a complicated topic on which entire books have been written. Some libraries, such as [boost](https://www.boost.org/) and [FFTW](http://fftw.org/) are well-known for complicated template usage.

If you find yourself intrigued by templates, I recommend [D](https://dlang.org) as having a richer, more programmer-friendly template mechanism than C++.
{/}

## Operator Overloading and Friends

C++ recognizes that there is no magic to operators, and it is awkward to write your matrix class with `x.add(y)` when you write your numbers as `x + y` instead, so it lets you overload operators.
In essence, this lets you add, to any class you want to work with, functions that are the implementation of operators. This lets you write math with your custom matrix class just like you do with the built-in number types.

Once you have this, though, it becomes tempting to use in other ways.
One of the most famous is the `ostream` type, C's wrapper over file writing, which overloads the left-shift operator to mean "write".

{.example ...}
The following C++ code writes "Hello, world!" and then an end-of-line.
There are no integers being shifted; `<<` means "left-shift" for integers, but it means "here's some output" with ostreams like `cout`

```cpp
#include <iostream>
using namespace std;

int main() {
    cout << "Hello, world!" << endl;
}
```

This code also uses a namespace; namespaces are a way of avoiding name collisions,
and most of the common C++ libraries we'll use are in the `std` namespace.
We could have equivalently written it as

```cpp
#include <iostream>
int main() {
    std::cout << "Hello, world!" << std::endl;
}
```

`cout`, by the way, is C++'s `ostream` wrapper around file descriptor `1`,
just as `stdout` is C's `FILE *` wrapper around the same.
{/}

C++ also allows what it calls "`friend`" functions, which are used to add functions and operators to existing classes in new files.
A common example is to add `ostream << mytype` operators for new types.

# STL

The Standard Template Library (STL) is a collection of C++ types and functions that provide much of the programming ease that C++ offers over C. Although there are other components of it, the best known parts are a collections library.

## Pointer-based

- `std::list<T>` a doubly-linked list
- `std::map<K,V>` a tree map, with the $O(\log n)$ time and space that entails
- `std::set<T>` a tree set, with the $O(\log n)$ time and space that entails
- `std::multiset<T>` a tree multi-set (allows duplicates) with the $O(\log n)$ time and space that entails
- `std::multimap<K,V>` a tree multi-map (allows duplicates) with the $O(\log n)$ time and space that entails

## Array-based

- `std::vector<T>` a resizable array of `T`. Much like a Java `ArrayList`, it offers $O(1)$ access to any member, amortized $O(1)$ adding at back, and $O(n)$ inserting or removing in middle.
- `std::deque<T>` a resizable array of small fixed-size arrays of `T` designed so that space is $O(n)$, access to front and back (for add and remove) is $O(1)$. The extra inner arrays add a little memory efficiency.

## Limited API

- `std::stack<T>` can be pushed and popped, FIFO-style, very efficiently, and nothing else.
- `std::queue<T>` can be pushed and popped, LIFO-style, very efficiently, and nothing else.

# Other common C++ library features

The standard C++ library contains several non-templated types too:

- `std:string` a smart string that overloads `+` to concatenate, stores length accessibly, etc
