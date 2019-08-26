---
title: Using SSH
...

SSH (the **S**ecure **SH**ell) is a protocol for allowing people to access a computer over the Internet and run programs on it as if they were physically present.

# Introductory Game

Visit <http://overthewire.org/wargames/> and read.
Many of the pages list web resources to help you learn more.

If you don't like reading (☹), try

````bash
$ ssh bandit0@bandit.labs.overthewire.org -p 2220
````

and consult <http://overthewire.org/wargames/bandit/bandit0.html> to get started.

# Software

If you are running      Then you should use
----------------------  -----------------------------
Linux                   open a terminal and use the built-in `ssh` command
MacOS                   open the `Terminal` app, then use the built-in `ssh` command
Windows 10              [enable](https://devblogs.microsoft.com/powershell/using-the-openssh-beta-in-windows-10-fall-creators-update-and-windows-server-1709/) it once; then open the `PowerShell` app and use the `ssh` command
Haiku                   open a terminal and use the built-in `ssh` command
FreeBSD                 open a terminal and use the built-in `ssh` command
OpenBSD                 open a terminal and use the built-in `ssh` command
Irix                    open a terminal and use the built-in `ssh` command

# Using key pairs instead of passwords

Typing passwords is both less secure (key-sniffers, typos, typing wrong passwords, etc) and more tedious than using a private key.

## Concept

You'll place a file on your computer and a file on the remote computer.
They are matched, and each provides half of the work needed to do a job.
When you log in, the remote computer will do half the work with its file, then send that to your computer to do the other half and return^[This is a gross over-simplification, but gets the core idea across. We'll see what's really going on in COA2].

## Practice

The following commands should work on any system with SSH installed,
with appropriate changes to `username@username@the.server.edu`;
if on Windows, you also have to use `\` instead of `/`:

```bash
ssh-keygen -f ~/.ssh/id_rsa
ssh-copy-id -i ~/.ssh/id_rsa username@username@the.server.edu
```

It should be safe to accept the defaults on each prompt except the password prompts.
Note the password prompts will accept what you type but not display it.

# Short summary

Interactive Terminal
:   Open with `ssh username@the.server.edu`
    
    Close with `exit` or Ctrl+D
    
    {.example} `ssh mst3k@portal.cs.virginia.edu`

Run remote command and see output
:   `ssh username@the.server.edu command arg1 arg2 ...`

    --*or*--
    
    `ssh username@the.server.edu "commands to execute"`

    {.example} `ssh mst3k@portal.cs.virginia.edu ls -l`
    
    {.example} `ssh mst3k@portal.cs.virginia.edu "cd /tmp; ls"`

Copy files from local to remote
:   `scp file file2 file3 ... user@the.server.edu:path/to/destination/`

    {.example} `scp testfile.c mst3k@portal.cs.virginia.edu:code/demo1/`
    
    Note that `scp` will not create directories, but `ssh` can:
    
    {.example} `ssh mst3k@portal.cs.virginia.edu mkdir -p code/demo1/`

Copy files from remote to local
:   `scp user@the.server.edu:path/to/source/filename path/to/destination/`
    
    (use `./` for "put it where I am")

    {.example} `scp mst3k@portal.cs.virginia.edu:code/demo1/testfile.c ./`


# Example for remote builds of local files

```bash
scp myfile1.c myfile2.c mst3k@portal.cs.virginia.edu:project1/
ssh mst3k@portal.cs.virginia.edu "cd project1/; clang *.c; ./a.out"
```
