---
title: C++ Inheritance
...

This page is intended to be a rapid overview of how OO features like inheritance and polymorphism can be implemented in C++.

# Running example

We'll use the following toy example throughout this writeup:

````cpp
#include <stdio.h>

class base {
public:
    double length; /// in meters
    double ftLength() { return length / 0.3048; } /// in feet

    base(double len) { this->length = len; } // constructor

    virtual double size() { return length; }
};

class derived : public base {
public:
    double height; /// in meters
    double ftHeight() { return height / 0.3048; } /// in feet

    derived(double len, double hei) : base(len) { this->height = hei; } // constructor

    virtual double size() { return length * height; }
    virtual int dimensions() { return 2; }
};

int main() {
    base *b = new base(3);
    base *d = new derived(4, 5);
    printf("lengths:  %g and %g\n", b->length, d->length);
    printf("imperial: %g and %g\n", b->ftLength(), d->ftLength());
    printf("sizes:    %g and %g\n", b->size(), d->size());
}
````

This is arguably a bad example *from an OO standpoint* (we made the fields public, and `size` doesn't even pass dimensional analysis) and a C++ standpoint (I wrote the constructors without initializer lists and use `printf` instead of `cout`),
but it will serve us well from a "how C++ works" standpoint.

Just for reference, the code prints

    lengths:  3 and 4
    imperial: 9.84252 and 13.1234
    sizes:    3 and 20

# Storing attributes

Note that we have declared both `b` and `d` to be `base *`,
even though `d` actually points to a `derived`.
This is handled by the simple expedient of laying out each `derived` with the attributes of `base` in the front:

Offset          Base        Derived     `b`         `d`
-------------   ----------  ----------  ----------  ----------
0--7            *vtable*    *vtable*    *vtable*    *vtable*
8--15           `length`    `length`    3.0         4.0
16--23          (nothing)   `width`                 5.0

: Memory layout of objects


Thus, when we assemble our code and `d->length`{.cpp} turns into something like `n(%rdx)` where `%rdx` contains value of `d`,
we arrive at the `length` attribute as if `d` was a `base *`, with no extra work needed.

The special *vtable* entries above will are explained in more detail under [Virtual functions].

# Non-virtual functions

Non-virtual functions like `ftLength` are turned into a single global function,
just like any other function
but with a first parameter in the assembly of `base *this` inserted automatically by the compiler.
Thus the assembly for `ftLength` looks like

    movsd   8(%rdi), %xmm0
    divsd   const3048(%rip), %xmm0
    ret

In other words, `%rdi` (the first argument) is the `base *this`{.cpp},
`length` is at offset `8` and moved into one of the floating-point registers (`%xmm0`),
the bytes of the constant `0.3048` are stored instruction-pointer-relative label `const3048` and used to divide,
which leaves the result in the correct register to return.
Because `length` is at the same offset for both `base` and `derived`, this works for both `base *this`{.cpp} and `derived *this`{.cpp}.

# Virtual functions

Non-virtual functions are stored in the global address space, and thus cannot be overridden in subclasses.
C++ allows overloading by using *virtual* functions.

Conceptually, in C++ every class's first member is a pointer to a `struct` called a **vtable** with only function-pointer attributes.
There's just one copy of each such `struct` in memory, stored in the read-only globals.
The one for `base` looks like

Offset          Member name     Points to
-------------   --------------  ------------------------
0--7            `size`          `base::size` 

: `base`'s `vtable`

and the one for `derived` looks like

Offset          Member name     Points to
-------------   --------------  ------------------------
0--7            `size`          `derived::size` 
8--15           `dimensions`    `derived::dimensions` 

: `derived`'s `vtable`

Thus there are three functions in the assembly (`base::size`, `derived::size`, and `derived::dimensions`)
and each object has one extra attribute:

Offset          `b`                         `d`
-------------   --------------------------- ------------------------------
0--7            pointer to `base`'s vtable  pointer to `derived`'s vtable
8--15           3.0                         4.0
16--23                                      5.0

: Memory layout of objects

The compiler then translates `d->size()`{.cpp} into the equivalent of what in C we might write as `d->vtable->size(d)`{.c}

{.aside ...} I'm not sure the above is the best explanation.
I wrote a different one too, a portion of which I've included here in case you prefer it.

In C' we'd write the virtual functions and vtables as something like

````c
/// the three functions
double base_size(base *this) { return this->length; }
double derived_size(derived *this) { return this->length * this->width; }
int derived_dimensions(derived *this) { return 2; }

/// base
struct base;
struct derived;

/// the three functions
double base_size(struct base *this);
double derived_size(struct derived *this);
int derived_dimensions(struct derived *this);

/// base
struct  {
    double (*size)(struct base *);
} base_vtable = { base_size };

typedef struct base {
    void **vtable;
    double length;
} base;

base *base_constructor(double len) {
    base *ans = malloc(sizeof(base));
    ans->vtable = (void *)&base_vtable;
    ans->length = len;
    return ans;
}

/// derived
struct {
    double (*size)(struct derived *);
    int (*dimensions)(struct derived *);
} derived_vtable = { derived_size, derived_dimensions };

typedef struct derived {
    void **vtable;
    double length;
    double width;
} derived;

derived *derived_constructor(double len, double wid) {
    derived *ans = malloc(sizeof(derived));
    ans->vtable = (void *)&derived_vtable;
    ans->length = len;
    ans->width = wid;
    return ans;
}

double base_size(struct base *this) { return this->length; }
double derived_size(struct derived *this) { return this->length * this->width; }
int derived_dimensions(struct derived *this) { return 2; }


/// how we call virtual functions
double invoke_size(base *b) {
    //       cast as function   1st vtable entry   argument
    return ((double (*)(base *)) (b->vtable[0]))      (b);
}
````

The above basically works, but I found trying to write why (and handle the ugliness of function pointer syntax) painful,
so I re-wrote it with the main test of this page.
{/}

