---
title: Debugger example
...


This file is a walk-through of locating and fixing a bug
using `lldb` and `ghex` (a hex editor)
to fix [`cmdadd`](files/cmdadd).
You may find it useful to follow along on your own.

The `cmdadd` program is *supposed* to add all of its command-line arguments, so that `./cmdadd 2 3 5 8` should display `Sum: 18`. We start by verifying it does not work: it displays `Sum: 36` instead.
So we need to debug it.

1. We start by invoking `lldb cmdadd`

    We see lldb tell us it loaded correctly:
    
        (lldb) target create "cmdadd"
        Current executable set to 'cmdadd' (x86_64).

2. Let's try the same invocation we had earlier: `run 2 3 5 8`

    lldb gives us the same output running `cmdadd` directly did:

        Process 10873 launched: '/home/mst3k/cmdadd' (x86_64)
        Sum: 36
        Process 10873 exited with status = 0 (0x00000000) 
    
    The two "process" lines tell us the program started and ended;
    the "exit status" is the number returned by the `main` function,
    where `0` is a program's way of saying "everything went OK"

3. Let's look at the disassembled binary.
    
    We start by trying just `di -f` but it does not work.
    We can type `di sdfsdgfdsfgs` or some other gibberish to get lldb to list `di` options, or we can look in the table above to find other uses.
    
    We know every program starts in `main`, so we try `di -n main`.
    This shows us some assembly; we look in particular for `callq` to other functions, as `main` rarely does all the work itself.
    There are three such lines:
    
        0x5555555551f8 <+72>:  callq  0x1040                    ; symbol stub for: atoll
        0x555555555204 <+84>:  callq  0x1150                    ; add
        0x555555555228 <+120>: callq  0x1030                    ; symbol stub for: printf

    lldb gives us comments after the `;`.
    The "symbol stub"s are to built-in functions, which you can learn more about
    by opening a new terminal window and typing `man 3 atoll` or `man 3 printf`.
    However, since these are built-in they are not likely to be the problem.

    Let's look at the other function, `add`.
    We type `di -n add` and see the following:

        cmdadd`add:
        0x555555555150 <+0>:  pushq  %rbp
        0x555555555151 <+1>:  movq   %rsp, %rbp
        0x555555555154 <+4>:  subq   $0x20, %rsp
        0x555555555158 <+8>:  movq   %rdi, -0x10(%rbp)
        0x55555555515c <+12>: movq   %rsi, -0x18(%rbp)
        0x555555555160 <+16>: cmpq   $0x0, -0x18(%rbp)
        0x555555555165 <+21>: jne    0x1178                    ; <+40>
        0x55555555516b <+27>: movq   -0x10(%rbp), %rax
        0x55555555516f <+31>: movq   %rax, -0x8(%rbp)
        0x555555555173 <+35>: jmp    0x1197                    ; <+71>
        0x555555555178 <+40>: movq   -0x10(%rbp), %rax
        0x55555555517c <+44>: addq   $0x2, %rax
        0x555555555180 <+48>: movq   -0x18(%rbp), %rcx
        0x555555555184 <+52>: subq   $0x1, %rcx
        0x555555555188 <+56>: movq   %rax, %rdi
        0x55555555518b <+59>: movq   %rcx, %rsi
        0x55555555518e <+62>: callq  0x1150                    ; <+0>
        0x555555555193 <+67>: movq   %rax, -0x8(%rbp)
        0x555555555197 <+71>: movq   -0x8(%rbp), %rax
        0x55555555519b <+75>: addq   $0x20, %rsp
        0x55555555519f <+79>: popq   %rbp
        0x5555555551a0 <+80>: retq   

4. We could try to understand what this code is doing,
    but it can be easier to see it in action.
    We start by adding a break point: `br set -n add`

        Breakpoint 1: where = cmdadd`add, address = 0x0000555555555150

    We then `run 2 3 5 8` to reach that breakpoint
    
        Process 10990 launched: '/home/mst3k/cmdadd' (x86_64)
        Process 10990 stopped
        * thread #1, name = 'cmdadd', stop reason = breakpoint 1.1
            frame #0: 0x0000555555555150 cmdadd`add
        cmdadd`add:
        ->  0x555555555150 <+0>: pushq  %rbp
            0x555555555151 <+1>: movq   %rsp, %rbp
            0x555555555154 <+4>: subq   $0x20, %rsp
            0x555555555158 <+8>: movq   %rdi, -0x10(%rbp)

    Notice llbd disassembles the local context for us.

5. Let's see what else is in scope.
    We know which registers are used to pass arguments, so let's list them with `reg read rdi rsi rdx rcx r8 r9`
    
         rdi = 0x0000000000000000
         rsi = 0x0000000000000002
         rdx = 0x0000000000000000
         rcx = 0x1999999999999999
          r8 = 0x00007fffffffe2d2
          r9 = 0x0000000000000000

    So we can see that `add` was invoked as `add()` or `add(0)` or `add(0, 2)` or `add(0, 2, 0)` or `add(0, 2, 0, 0x1999999999999999)` or ...
    We'll be able to figure out which one by seeing which registers are read before being set inside the program.
    
6. Let's move step-by-step.

    Each time we type `step` it executes the instruction previously pointed to
    and shows us the next several instructions on deck.
    
        (lldb) step
        Process 10990 stopped
        * thread #1, name = 'cmdadd', stop reason = instruction step into
            frame #0: 0x0000555555555151 cmdadd`add + 1
        cmdadd`add:
        ->  0x555555555151 <+1>:  movq   %rsp, %rbp
            0x555555555154 <+4>:  subq   $0x20, %rsp
            0x555555555158 <+8>:  movq   %rdi, -0x10(%rbp)
            0x55555555515c <+12>: movq   %rsi, -0x18(%rbp)
    
    If we type `step` several more times, we eventually reach
    
        (lldb) step
        Process 10990 stopped
        * thread #1, name = 'cmdadd', stop reason = instruction step into
            frame #0: 0x0000555555555160 cmdadd`add + 16
        cmdadd`add:
        ->  0x555555555160 <+16>: cmpq   $0x0, -0x18(%rbp)
            0x555555555165 <+21>: jne    0x555555555178            ; <+40>
            0x55555555516b <+27>: movq   -0x10(%rbp), %rax
            0x55555555516f <+31>: movq   %rax, -0x8(%rbp)

    Up to this point we've moves `%rsp`, copied it into `%rbp`,
    and moved two registers into that memory: `%rdi` into `-0x10(%rdp)`
    and `%rsi` into `-0x18(%rdp)`.
    That implies we have a two-argument function (just `rdi` and `rsi` were accessed)
    and this function was invoked as `add(0, 2)`.
    
7.  We're now about to run a `cmpq` and a `jne`;
    that is, to compare to values and jump if they are not equal.
    
        if (value1 != value2) goto somewhere
    
    One of the values is 0; the others is `-0x18(%rbp)`.
    Just for practice, let's see what is in `-0x18(%rbp)`.
    That's a memory address, so we need a `memory read`;
    it's being compared with `cmpq` so we need `-s 8` and `-c 1`;
    but what is the address? to figure that out we find our what's in `%rbp`
    
        (lldb) register read rbp
             rbp = 0x00007fffffffde40
    
    and add `-0x18` to it to get `0x7fffffffde28`,
    then use this in a memory read:
    
        (lldb) me rea -s8 -c1 -fx 0x7fffffffde28
        0x7fffffffde28: 0x0000000000000002
    
    (note: we could also have done `me rea -s8 -c1 -fx 0x00007fffffffde40-0x18` to get the same result)
    
    So we are comparing `2` to `0` and jumping if not equal, so we expect to jump.
    Let's verify this:
    
        (lldb) step
        Process 11266 stopped
        * thread #1, name = 'cmdadd', stop reason = instruction step into
            frame #0: 0x0000555555555165 cmdadd`add + 21
        cmdadd`add:
        ->  0x555555555165 <+21>: jne    0x555555555178            ; <+40>
            0x55555555516b <+27>: movq   -0x10(%rbp), %rax
            0x55555555516f <+31>: movq   %rax, -0x8(%rbp)
            0x555555555173 <+35>: jmp    0x555555555197            ; <+71>
        (lldb) step
        Process 11266 stopped
        * thread #1, name = 'cmdadd', stop reason = instruction step into
            frame #0: 0x0000555555555178 cmdadd`add + 40
        cmdadd`add:
        ->  0x555555555178 <+40>: movq   -0x10(%rbp), %rax
            0x55555555517c <+44>: addq   $0x2, %rax
            0x555555555180 <+48>: movq   -0x18(%rbp), %rcx
            0x555555555184 <+52>: subq   $0x1, %rcx

8.  `step`ping some more we see that we are
    
    a. loading the first argument in `rax`
    a. adding 2 to it
    a. loading the second argument into `rcx`
    a. subtracting 1 from it
    a. loading `rax` into `rdi`, the 1^st^ argument spot for a call
    a. loading `rax` into `rsi`, the 2^nd^ argument spot for a call
    a. calling `<+0>`: that is, this very function, `add`.
    
    So we have a recursive function; `add(0, 2)` invoked `add`; let's see what it's arguments will be:
    
        (lldb) reg read rdi rsi
             rdi = 0x0000000000000002
             rsi = 0x0000000000000001

9.  We're in a recursive function, and have a breakpoint at its beginning,
    so we can repeatedly type `continue` and `register read rdi rsi` to see what arguments it has each time it is invoked.
    
    Tracking these, we see (removing some messages for brevity)

        (lldb) continue
        (lldb) reg read rdi rsi
             rdi = 0x0000000000000004
             rsi = 0x0000000000000000
        (lldb) continue
        (lldb) reg read rdi rsi
             rdi = 0x0000000000000004
             rsi = 0x0000000000000003
        (lldb) continue
        (lldb) reg read rdi rsi
             rdi = 0x0000000000000006
             rsi = 0x0000000000000002
        (lldb) continue
        (lldb) reg read rdi rsi
             rdi = 0x0000000000000008
             rsi = 0x0000000000000001
        (lldb) continue
        (lldb) reg read rdi rsi
             rdi = 0x000000000000000a
             rsi = 0x0000000000000000
        
        ...

    Combining this with what we've already seen, we have the following sequence of calls:
    
    a. `add(0, 2)`
    a. `add(2, 1)`
    a. `add(4, 0)`
    a. `add(4, 3)`
    a. `add(6, 2)`
    a. `add(8, 1)`
    a. `add(10, 0)`
    a. `add(10, 5)`
    a. `add(12, 4)`
    a. `add(14, 3)`
    a. `add(16, 2)`
    a. `add(18, 1)`
    a. `add(20, 0)`
    a. `add(20, 8)`
    a. `add(22, 7)`
    a. `add(24, 6)`
    a. `add(26, 5)`
    a. `add(28, 4)`
    a. `add(30, 3)`
    a. `add(32, 2)`
    a. `add(34, 1)`
    a. `add(36, 0)`
    
10. Looking at the above, I notice we seem to be counting down each of the arguments, and counting up by twos.
    If all of the counting up steps were by 1 instead, we'd be OK.
    So I need to figure out where the down-by-2 is in the binary:
    `di -n add -b` includes this line
    
        0x55555555517c <+44>: 48 83 c0 02        addq   $0x2, %rax

11. Let's fix that. I wanted `01` not `02`.
    So I open up a hex editor, such as `ghex cmdadd`
    In that, I find the instruction I don't want by
    
    a. Ctrl+F to open the find dialog
    b. Type `48 83 c0 02`
    c. Find Next
    d. Verify that there is only one instruction with this encoding
        by pressing "Find Next" again.
        If there were more than one, we'd pick a larger string,
        perhaps including the bytes of the `movq` on either side.
        
        Note this is not at byte `0x55555555517c` of the executable.
        The loader relocates these bytes when we run the program.
    e. Edit the incorrect byte (changing the `02` to `01`) and save the file

12. Test the fix: `./cmdadd 2 3 5 8` should now display `Sum: 18`.
