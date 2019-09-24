---
title: Tools Help
...

We were assured the systems would work so that `ssh` to `portal` would solve all our future needs. Reports from early labs suggest that is not the case. Hence, this lab is designed to make sure each of you have an alternative setup you could use.

You are encouraged to do this lab on your own at home, then just quickly check it off with a TA.

{.exercise ...} To complete this lab, do both of the following:

1. Show a TA you can [use `ssh` and `module load`](#how-to-on-portal) to follow the instructions in the section [Show at least this](#show-at-least-this) below on `portal.cs.virginia.edu`.
1. Show a TA you can use a [virtual box](#how-to-with-a-virtual-machine) or [other system](#how-to-on-your-own) on your own computer to follow the instructions in the section [Show at least this](#show-at-least-this) below on your own computer.

{/}

# Show at least this

1. Make and enter a directory named `tools_test`
2. Create a file there^[You learned how to do this in [Lab 00](lab00-ssh-ed.html#cli-editor) using one of Nano, Emacs, or VIM. If you did not, you needed too; please go back and do that part of Lab 00 now!] named `hello.c` which contains the following C code.
    
    ````c
    #include <stdio.h>
    int main() {
        puts("This file shows your C toolchain is working");
    }
    ````

    We will explain this code later, but want to make sure you have C working first.

3.  Run the following and show the output to a TA.
    Several of these commands will display things,
    and the last two (`run` and `quit`) will have a different-looking prompt than the others.
    Both `./a.out` and `run` should have, as part of their output, "This file shows your C toolchain is working".
    
    ````bash
    pwd
    clang hello.c
    ./a.out
    lldb a.out
    run
    quit
    ````

# How-to on `portal`

You've seen how to `ssh` into `portal.cs.virginia.edu` before.
Some things you should know for this:

You have to load modules
:   The CS servers hide most programs until you ask for them.
    You ask for them by running `module load` and then the module you want to access the programs inside of.
    The modules we'll need in this class are
    
    - `module load clang-llvm`
    - `module load nano` (or `module load emacs` if you prefer `emacs`, or no module needed if you prefer `vim`)

Other than loading these modules after you `ssh` in to `portal` (each time you `ssh` into portal), you should be able to use the skills you learned in [Lab 00](lab00-ssh-ed.html) and [Lab 01](lab01-git-infotheory.html) to complete the [Show at least this](#show-at-least-this) material.

# How-to with a virtual machine

A virtual machine is a program that pretends to be an entire computer, so you can run a different operating system inside your current operating system. We recommend using this to get Linux running inside your MacOS or Windows environment.

1. Download and install VirtualBox from <https://www.virtualbox.org/wiki/Downloads>

2. Download our (1.8 GB) VirtualBox OVA file either from [UVA Box](https://virginia.box.com/s/b1nhtc3z2uuhze5xxjlcjh8gjb9wx6wy) or from [Collab](https://collab.its.virginia.edu/access/content/group/376189b0-ab8a-4906-a181-153ed4ffaf4c/COA_Lubuntu.ova) (we have two copies available, either one is fine)

3. If you are running Windows,
    a. Disable Windows Hyper-V (an alternative virtualization layer which is incompatible with VirtalBox) by running the following from powershell:
        
            bcdedit /set hypervisorlaunchtype off
    
    b. You can re-enable Hyper-V by running the following from powershell; note though that VirtualBox requires it to be disabled, so don't do this if you still need VirtalBox:
        
            bcdedit /set hypervisorlaunchtype on

4. Run VirtualBox, and within it initialize your virtual machine as follows
    a. In the menu bar, click File → Import Appliance…
    b. Click the folder icon on the right of the text box and browse to where you downloaded the OVA file in step 2 above
    c. Click Next, then Import
    d. Wait for it to finish

5. To run your virtual machine,
    a. Run VirtualBox
    b. Select "UVA_COA_Lubuntu" on the left panel
    c. Click "Start" 


# How-to on your own

If you run a mostly POSIX-compliant operating system (Linux, FreeBSD, OpenBSD, AIX, Solaris, and almost all the others with one notable exceptions), you can probably get `clang`, `git`, and `lldb` to work with no extra effort.

As a special case, macOS is POSIX-compliant but by default does not include most of the tools we'll need, has a slightly different take on some parts of C than normal, and often has commands hidden under non-standard names or the like. If you install `clang` and `lldb` through the macOS developer tools, you can probably use your macOS machine directly with no virtual machine. You are welcome to do this, but note that we may not be able to help if something goes wrong.

If you run Windows, there's a thing called the Linux Subsystem for Windows which can let you make windows (almost) act like a POSIX-compliant OS. You are welcome to do this, but note that we may not be able to help if something goes wrong.

