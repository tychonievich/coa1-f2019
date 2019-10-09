---
title: Assembly
...

# Introduction

In this assignment you need to write x86-64 assembly by hand.
This is not something most programmers do often, but when it comes up in the workplace
being one of the few there who can do it will help your company and make you shine.

Write your code in a file named `matlib.s`.
We provide a base file that includes the input and output interaction (in `main` and `printNum`) to test the code you will write: [`matlib.s`](files/matlib.s).
Write your code where the comments say "TO DO: write this function".

# Logistics

## Writing your code 

You can edit your assembly however you wish, but we *strongly* recommend [a CLI editor](lab00-ssh-ed.html#cli-editor).

{.aside ...}
Why a CLI editor?

It is common to interact with servers that do not have their own monitors.
In these cases, you typically attach to the server via `ssh`
and have access only to a terminal, not a full windowing environment.
The more comfortable you are with doing common programming tasks in the terminal,
the better these experiences will be.
{/}

## Testing your code

We recommend using `git` to version and test your code.

1. Use a git project for this PA and check it out on both your laptop and the server.
    You can use the same project you made in [Lab 01](lab01-git-infotheory.html#creating-a-project) or a new one for just this PA
2. Download [the starter code](files/matlib.s) into that project directory on your laptop
    and `git add matlib.s` to tell `git` to track it
3. Edit locally, and when you want to test it
    a. commit your edits (e.g. `git commit -a -m 'I think I got product working now'`)
    a. push your committed edits (e.g. `git push`)
    a. tell the server to pull, compile, and run with `ssh`
        i. `ssh mst3k@portal.cs.virginia.edu`
        ii. `cd coa1-code; git pull; clang matlib.s && ./a.out`
        iii. `exit`

<!--
        - a one-liner for this is `ssh mst3k@portal.cs.virginia.edu 'cd coa1-code; git pull; clang matlib.s && ./a.out'`
        - remember, up-arrow can re-enter previous commands
    a. You can also put all these commands in a file (`pa06test.sh` on MacOS, `pa06test.ps1` on Windows) and run them at once (using `bash pa06test.sh` on MacOS, `pwsh pa06test.ps1` on Windows)
        
        ````bash
        git commit -a -m 'auto-commit before testing'
        git push
        ssh mst3k@portal.cs.virginia.edu 'module load llvm-clang; cd coa1-code; git pull; clang matlib.s && ./a.out'
        ````
-->

This way you'll both (a) have a backup of every version you test, (b) have a copy on the server in case something happens to your laptop, and (c) know you're using the same compiler, processor variant, etc, that we are.

If you get into an infinite loop, Ctrl+C is your friend.

# Write two functions

In AT\&T syntax x86-64 assembly, write two routines:
one that computes the product of two numbers, and one that computes the power of two numbers.

It should be possible to assemble your code by typing `clang matlib.s`
to generate a file, `a.out`, that can be run by by typing `./a.out`.
When run, it should^[Along the way, you might accidentally write code that runs forever. If the program freezes while running, try pressing Ctrl+C to interrupt it.] ask for two integers and display their product and exponent.

## Product

The first subroutine, `product`,
should compute and return the product of the two integer arguments.
It **must not use** multiplication or division instructions.
It must compute this **iteratively**, not recursively.

{.example ...}
There are other correct solutions,
but a simple one might follow an approach like the following pseudo-code:

    function product(x, y)
        z = 0
        while y > 0 repeat
            z += x
            y -= 1
        end while
        return z
    end function

There also exists a much more efficient solution that uses bit shifts.
{/}

You may assume that both of the parameters are positive integers
(i.e., neither negative or zero)
and that the result can fit in a single 64-bit register.

## Power

The second subroutine, `power`,
should compute and return its first argument raised to the power of its second argument.
It **must use** the `product` routine you wrote to do this,
not x86-64 multiplication instructions or other routines you did not write.
It must compute this **recursively**, not iteratively.

{.example ...}
There are other correct solutions,
but a simple one might follow an approach like the following pseudo-code:

    function power(x, y)
        if y == 1 then
            return x
        else
            return product(x, power(x, y-1))
        end if
    end function
{/}

You may assume that both of the parameters are positive integers
(i.e., neither negative or zero)
and that the result can fit in a single 64-bit register.


# Submit

Submit your assembly as `matlib.s`.
