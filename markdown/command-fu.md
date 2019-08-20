---
title: Command-fu
...

This is a place to record various command-line tricks and options I use in class.
Due to a series of mis-translations involving 功夫 and Hollywood stereotypes,
the "magic you can do on the command line" is traditionally called command-fu, command-line fu, script-fu, etc.

# 2018-10-15

-   `clear`{.bash} scrolls so the next line is the top line of the terminal window.
    It does not remove anything, just scrolls.

-   `cat hello.c`{.bash} dumps the contents of `hello.c` to the terminal window.

-   `clang -S hello.c`{.bash} the `-S` flag means "I want an assembly file, not a program"
    and generates `hello.s` instead of `a.out`.
    
    I also mis-typed `-s` instead of `-S` once. That means "strip" -- i.e., remove all symbol table information from the binary (making it hard to link with other files and debugging hard, but the resulting file a bit smaller).

-   `cpp hello.c`{.bash} runs the C pre-processor on `hello.c`, outputting the resulting `.c` file to the terminal

-   `cpp hello.c | less`{.bash} runs the C pre-processor on `hello.c`, outputting the resulting `.c` file into the `less` program so we can scan through it.
