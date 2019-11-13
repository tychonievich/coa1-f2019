---
title: Linked List
...

Write a C file named `linkedlist.c` that contains implementations of the following functions for manipulating a doubly-linked list.

Your code should begin with `#include "linkedlist.h"`{.c}, where [linkedlist.h may be downloaded here](files/linkedlist.h).
We will assume you use this exact `linkedlist.h` in our testing: if you change it, we will not use those changes in our compilation and testing of your code.
Comments in `linkedlist.h` also describe what each function is supposed to do.

To get you started, here is one of the functions you are supposed to write,
fully and correctly implemented for you; feel free to copy-paste it into your `linkedlist.c` file:

````c
/* reference solution provided with assignment description */
void ll_show(ll_node *list) {
    ll_node *ptr = ll_head(list);
    putchar('[');
    while(ptr) {
        if (ptr->prev) printf(", ");
        if (ptr == list) putchar('*');
        printf("%d", ptr->value);
        if (ptr == list) putchar('*');
        ptr = ptr->next;
    }
    puts("]");
}
````

# Example testing code

Once everything is implemented correctly, this code:

````c
int main(int argc, const char *argv[]) {
    ll_node *x = NULL;
    putchar('A'); ll_show(x);
    x = ll_insert(25, x, 1);
    putchar('B'); ll_show(x);
    x = ll_insert(1, x, 0);
    putchar('C'); ll_show(x);
    x = ll_insert(99, x, 1);
    putchar('D'); ll_show(x);
    x = ll_insert(0, x, 1);
    putchar('E'); ll_show(x);
    x = ll_insert(-8, ll_tail(x), 0);
    putchar('F'); ll_show(x);
    ll_node *y = ll_head(x);
    putchar('G'); ll_show(y);
    printf("Length: %lu (or %lu)\n", ll_length(y), ll_length(x));
    x = ll_remove(x);
    putchar('H'); ll_show(x);
    putchar('I'); ll_show(y);
    x = ll_remove(ll_find(y, 99));
    putchar('J'); ll_show(x);
    putchar('K'); ll_show(y);
    
    return 0;
}
````

should display

    A[]
    B[*25*]
    C[25, *1*]
    D[25, *99*, 1]
    E[25, *0*, 99, 1]
    F[25, 0, 99, 1, *-8*]
    G[*25*, 0, 99, 1, -8]
    Length: 5 (or 5)
    H[]
    I[*25*, 0, 99, 1]
    J[25, 0, *1*]
    K[*25*, 0, 1]

You should probably also manually test all of the corner cases (`NULL`{.c} arguments; first-, last-, and middle-node arguments, finding values not in the list, etc).

# Tips

1. I have yet to see a student implement their first correct linked-list without drawing box-and-arrow pictures.

2. Debugging can be easier if you can see all of the pointers involved.
    An example function to do that is

    ````c
    /**
     * Debugging display: shows the address and all the fields of the node,
     * optionally with the nodes before (if b is true) and after (if f is true).
     *
     * Written by Luther Tychonievich and made available to all students.
     */
    void ll_dump(ll_node *list, int f, int b) {
        if (b && list->prev) ll_dump(list->prev, 0, 1);
        printf("%p: %p\t%d\t%p\n", list, list->prev, list->value, list->next);
        if (f && list->next) ll_dump(list->next, 1, 0);
    }
    ````

3. Don't [use after free](memory.html#use-after-free)!
    We will test your code with [the address sanitizer](memory.html#using-the-address-sanitizer) enabled
    and may also manually check for additional [memory errors](memory.html#common-kinds-of-problems).

