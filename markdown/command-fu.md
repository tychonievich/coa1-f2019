---
title: Command-fu
...

This is a place to record various command-line tricks and options I use in class.
Due to a series of mis-translations involving 功夫 and Hollywood stereotypes,
the "magic you can do on the command line" is traditionally called command-fu, command-line fu, script-fu, etc.

-   `clear`{.bash} scrolls so the next line is the top line of the terminal window.
    It does not remove anything, just scrolls.

-   `cat hello.c`{.bash} dumps the contents of `hello.c` to the terminal window.

-   `clang -S hello.c`{.bash} the `-S` flag means "I want an assembly file, not a program"
    and generates `hello.s` instead of `a.out`.
    
    Note the `-S` is capital; a lower-case `-s` means "strip" -- i.e., remove all symbol table information from the binary (making it hard to link with other files and harder to debug, but making the file smaller.

-   `cpp hello.c`{.bash} runs the C pre-processor on `hello.c`, outputting the resulting `.c` file to the terminal

-   `cpp hello.c | less`{.bash} runs the C pre-processor on `hello.c`, outputting the resulting `.c` file into the `less` program so we can scan through it.

-   `clang -c hello.c`{.bash} the `-c` flag means "I want an object file, not a program"
    (i.e., stop before linking) 
    and generates `hello.o` instead of `a.out`.

-   `clang hello.c -o hello`{.bash} the `-o` flag means "name the output this instead of the default name"
    and generates `hello` instead of `a.out`.
    
    Warning: if you do something like `-o hello.c` is over-writes your source code with the executable.
    Use `-o` with care.

-   `whatis printf`{.bash} or `man -f printf`{.bash} lists the all of the manual pages that are titled "`printf`"

-   `apropos printf`{.bash} or `man -k printf`{.bash} searches for manual pages that reference "`printf`"

-   `which clang`{.bash} shows the full path to the executable that will be run if you type `clang`
