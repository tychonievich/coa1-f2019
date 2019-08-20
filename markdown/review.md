---
title: Review
...

Semester     Exam   Practice                            Actual                          Key
----------- ------- ----------------------------------  -----------------------------   --------------------------------
Fall 2018     1     [pdf](files/f2018e1practice.pdf)    [pdf](files/f2018e1real.pdf)    [html](files/f2018e1key.html)
Fall 2018     2     [pdf](files/f2018e2practice.pdf)    [pdf](files/f2018e2real.pdf)    [html](files/f2018e2key.html)
Fall 2018     3     [pdf](files/f2018e3practice.pdf)    [pdf](files/f2018e3real.pdf)    [html](files/f2018e3key.html)

The final exam will be about half material from exams 1 and 2 and about half new material. The practice exam only contains examples of the new material.

The final will include printed-out excerpts from manual pages.
These might include pages for functions you have not previously used.
It may also contain reference material on various assembly instructions, etc.

Some material from previous exams has "timed out;" for example, if we ask about our toy ISA on the final we'll provide a reminder of its relevant details as we hope you've re-used the memory that used to remember where the `icode` goes in the encoding, etc.

You are expected to know, without consulting any source,

Assembly
:   
    1. AT&T syntax, including all addressing modes (`$1`, `%rax`, and the various memory accesses like `(%rax)` through `foo(%rax, %rbx, 8)`)

    1. The special use of `%rsp` as the stack pointer

    1. The calling convention use of `%rax`, `%rdi`, and `%rsi` -- if others are needed, they will be provided on the exam

    1. The 2-, 4-, and 8-byte versions of each instruction (e.g., `movw`, `movl`, and `movq`) and the first eight registers (e.g., `%ax`, `%eax`, `%rax`)

    1. x86-64 assembly instructions `mov`, `add`, `xor`, `call`, `ret`, `lea`, `cmp`, `jmp`, and the signed conditional jumps (`jle` and so on)

C
:   
    1. The meaning of all operators, including `|` vs `||`, `.` vs `->`, `?:`, etc.
        - but you do *not* need to know the difference between prefix- and postfix-notation for `++` and `--`
        - nor do you need to know the precedence of non-arithmetic operators
            - but if you don't know precedence, you had better use parentheses!

    1. The correct signature for `main` (both with and without arguments)

    1. The behavior of library functions `malloc`, `free`, `realloc`, `open`, `close`

    1. The library function `read`, but just its common usage, not all the special cases

    1. The library functions `puts`, `printf`, `fopen`, `fclose`, and at least one reads-from-`FILE *` function

You are of course also expected to recall all the syntax and semantics details needed to read and write code in both AT&T x86-64 and C.
