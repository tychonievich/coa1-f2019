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

# STL and standard library

The C++ Standard Library has a variety of types and algorithms in it.
Some of the best known of these are part of the "Standard Template Library (STL)", and sometimes people use "STL" informally to mean the full C++ Standard Lbirary.
There is more in the library than we can fully cover, but the following will get you going:

|include|provides|STL?|description|
|:-|:-|:-:|:-|
|`<list>`|`std::list<T>`|STL|doubly-linked list; key functions `push_`/`pop_front`, `push_`/`pop_back`|
|`<map>`|`std::map<K,V>`|STL|tree-map; key function `[key]`|
|`<stack>`|`std::stack<T>`|STL|stack; key functions `push`, `pop`, `top`|
|`<queue>`|`std::queue<T>`|STL|queue; key functions `push` `pop`, `front`|
|`<vector>`|`std::vector<T>`|STL|resizable array; key functions `[index]`, `insert`, `push_`/`pop_back`|
|`<iterator>`|`std::iterator<T>`|STL|use as `for(it = c.begin(); it != c.end(); ++it)`{.cpp} for STL iterator `it` and collection `c`|
|`<string>`|`std::string`||Wraps a `const char *` with various useful methods (`find`, `replace`, `size`, `+=`, `==`, etc)|
|`<iostream>`|`std::istream` and `std:ostream`||C++ file handling; operator `<<` writes and `>>` reads; defines `cin`, `cout,` and `cerr`|
|`<sstream>`|`std::stringstream`||Lets you treat a string like a file|

There are many others not listed above; see [wikipedia](https://en.wikipedia.org/wiki/C%2B%2B_Standard_Library) for a reasonable overview.


# Your Task

{.exercise ...}
Implement a postfix calculator, like you did in [PA09](pa09-postfix.html) but this time use C++.

- You must use `cin` to read input, `>>` to parse numbers, `string` and `==` to find operators, and `stack<double>`{.cpp} as your stack.
    
- You must implement `ostream & operator<<(ostream & o, stack<double> s)`{.cpp} to display a stack, and end your program with `cout << my_stack;`{.cpp}.
    
We strongly recommend, but do not requite, using a two-step read: read a word from `cin` into a `string`,
then feed the `string` into a `stringstream` and use the stringstream to parse a number. However, there are other solutions and you do not have to do this if you do not want to.

We strongly recommend, but do not require, adding `using namespace std;` after your `#include`s so that you can refer to included types by name without prefixing `std::` before each.
{/}


The input processing cal look like a loop which, as long as `cin` is `.good()` (i.e., not closed and with no read errors)

1. `>>` a `string` from `cin`
1. make a `stringstream` and feed it a `.str(...)` to parse
1. `>>` a `double` from the `stringstream`
1.  if that `>>`ing `.fail()`s,
    
    a.  check if the `string` is `==` to an operator, and 
    
        i.  if so apply the operator (ending if there are fewer than 2 operands)
            
            otherwise, end the loop
        
        otherwise, push the `double`

Use a `stack<double>` as your stack. Note that `pop` returns `void`; you'll need to use `top` to retrieve a value before `pop`ing it off the stack.

Displaying a `stack` is not something the STL natively supports.
You'll need to implement your own `ostream & operator<<(ostream & o, stack<double> s)`{.cpp}
which repeatedly displays (with `<<`) the `top()` and then `pop()`s until the stack is `.empty()`.
Note that we pass in the ouput stream by reference (with `&`), but pass the stack by value.
Passing by value means passing a copy, so `pop`ing `s` does not `pop` the stack that was passed in, only the local copy of it that the `operator<<` function contains.
