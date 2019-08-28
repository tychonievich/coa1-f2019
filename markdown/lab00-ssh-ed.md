---
title: SSH and Editors
...

This pseudo-lab is to be done on your own prior to the first lab.
There is no check-off or other grade process for this pseudo-lab.

# SSH

SSH (**S**ecure **SH**ell) is a ubiquitous tool for running arbitrary programs on remote machines.
We will use it extensively this semester.
It comes bundled with a few related tools, like SCP (**S**ecure **C**o**P**y), that we will also use.

## Accessing the SSH tool

You'll need to know how to run SSH:

If you are running      Then you should use
----------------------  -----------------------------
Linux                   open a terminal and use the built-in `ssh` command
MacOS                   open the `Terminal` app, then use the built-in `ssh` command
Windows 10              [install](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse) OpenSSH Client (following the "from the Settings UI" directions) once; then open the `PowerShell` app and use the `ssh` command
Haiku                   open a terminal and use the built-in `ssh` command
FreeBSD                 open a terminal and use the built-in `ssh` command
OpenBSD                 open a terminal and use the built-in `ssh` command
Irix                    open a terminal and use the built-in `ssh` command

## Learn basic shell with a tutorial/game

SSH gives you a shell; shells have long been ubiquitous tools in all systems except Windows, and they are increasingly used in Windows too.
The shell is sometimes also called the "command line" or the "console".

Visit <http://web.mit.edu/mprat/Public/web/Terminus/Web/main.html>, a somewhat cheesy introduction to the basics of the command line. Explore it until you

- feel comfortable with the use of `ls`, `pwd`, `cd` (including `cd ..`), and less
- have learned about `mv` and `man`

There is a lot more you can do (creating a magic locker, explore a hidden tunnel, learn about `grep` and `rm`, etc.) but those are the most important basics.

{.aside ...} Avoiding excessive typing

While in a shell, there are several keys to make you life easier; the most important are

Up and Down
:   The up and down arrow keys navigate through a history of previously-typed commands. On some systems, page-up and page-down also navigate in large chunks. 

Tab
:  Pressing the tab key when the cursor is preceded by an incomplete word that can only be completed in one way will fill in the rest of the word.

    Pressing tab twice when the cursor is preceded by an incomplete word that can be completed in several ways lists all of the completions the command line knows about.
{/}

## Learning SSH with games/tutorials

Visit <http://overthewire.org/wargames/> and read.
Many of the pages list web resources to help you learn more.

If you don't like reading (☹), try

````bash
$ ssh bandit0@bandit.labs.overthewire.org -p 2220
````

and consult <http://overthewire.org/wargames/bandit/bandit0.html> to get started.

We suggest getting to level 4 of Bandit, though you might find other levels and games there interesting.

# CLI Editor

Sometimes you need to edit a file on a remote computer over SSH.
We'll see other ways of doing this as the semester progresses, but you'll often need to work in a command-line interface (CLI).

A text editor is a tool designed for editing text, only. They typically provide programming language syntax highlighting (i.e., strings show up in a different color from integers) and sometimes other customizable features, but typically do no have error checking, compiling, executing, and other features of an IDE.

There are three main CLI editors in common use today:

EMACS
:   This was one of the first widely-used powerful CLI editors (released 1976).
    It has a large set of features and an idiosyncratic set of commands, many of which involve pressing Ctrl+X followed by another Ctrl+something command.
    
    As EMACS has lost popularity in the past decade, more and more servers choose not to install it.

    EMACS comes with Linux, and can be downloaded for Windows and obtained via homebrew for MacOS: <https://www.gnu.org/software/emacs/download.html>

VI
:   Or more commonly the updated version VIM (1979) is probably the most widely used today.
    It has a large set of features and an idiosyncratic set of commands involving two modes:
    "normal" mode where virtually every letter and number key has a special meaning and often a special meaning in a sequence
    and "insert" mode where those same keys instead type.
    
    To begin learning VI, go to <https://www.openvim.com/> and follow along.
    Then SSH into any system that supports it (the wargames servers do, for example, though they do not support saving files), type `vim`, and try it out. You'll probably need a cheat-sheet to remember all the keys: <https://vim.rtorr.com/>.
    
    You can also (from SSH) running `vimtutor` instead if you want a different approach to learning VI.
    
    VI comes with Linux and MacOS, and can be downloaded for Windows: <https://www.vim.org/download.php>

GNU nano
:   A much simpler (i.e. both easier to learn and less powerful) editor with more traditional key commands and an on-screen summary of the most used commands.
    
    To learn `nano`, SSH into any system that supports it (the wargames servers do, for example, though they do not support saving files), type `nano`, and follow the on-screen instructions.
    
    Note that in the instructions `^X` means Ctrl+X and `M-X` means Alt+X (or Esc X if you are on MacOS and Alt does not work for you).
    
    Nano comes with Linux and an old version comes with MacOS;
    it can be downloaded for Windows (via the [chocolatey package manager](https://chocolatey.org/)) by typing the following two lines into the PowerShell app:
    
    1. `Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))`{.powershell}
    2. `choco install nano`{.powershell}

Become sufficiently familiar with at least one of these editors that you can open, modify, and save files.
