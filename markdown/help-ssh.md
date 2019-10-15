---
title: Using SSH
...

SSH (the **S**ecure **SH**ell) is a protocol for allowing people to access a computer over the Internet and run programs on it as if they were physically present.

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
When you log in, the remote computer will do half the work with its file, then send that to your computer to do the other half, then send it back, thus allowing both computers to be confident the other computer is who it says it is^[This is a gross over-simplification, but gets the core idea across. We'll see what's really going on when we discuss digital certificates in COA2].

## Setup

The following commands should work on any system with SSH installed,
with appropriate changes to `username@the.server.edu`;[^error]

```bash
ssh-keygen -f ~/.ssh/id_rsa -t rsa -b 2048
```

When prompted for a passphrase by `ssh-keygen`, just press enter without typing anything.

```
ssh-copy-id -i ~/.ssh/id_rsa.pub username@the.server.edu
```

When prompted for a passphrase by `ssh-copy-id`, use your UVA CS account password.


[^error]:
    If on Windows, you also may need to use `\` instead of `/` (whether you do or not depends on which command line tool you use).
    
    There is a slight chance that `~/.ssh` will not already exist. In that case `ssh-keygen` will fail; if you see such a failure you can fix it by running 

    ````bash
    mkdir ~/.ssh
    chmod 700 ~/.ssh
    ````

    and then re-run the above commands


### Multiple machines

You'll need to do the `ssh-keygen` once per client machine you use (e.g., you laptop)

# Short summary

{.example ...} The following copies files to the CS SSH portal, then compiles and runs them on the server and displays the output on your laptop.

```bash
scp myfile1.c myfile2.c mst3k@portal.cs.virginia.edu:project1/
ssh mst3k@portal.cs.virginia.edu "cd project1/; clang *.c; ./a.out"
```
{/}

Interactive Terminal
:   Open with `ssh username@the.server.edu`
    
    Close with `exit` or Ctrl+D
    
    {.example ...} `ssh mst3k@portal.cs.virginia.edu`
    {/}

Run remote command and see output
:   `ssh username@the.server.edu command arg1 arg2 ...`

    --*or*--
    
    `ssh username@the.server.edu "commands to execute"`

    {.example ...} `ssh mst3k@portal.cs.virginia.edu ls -l`
    {/}

    {.example ...} `ssh mst3k@portal.cs.virginia.edu "cd /tmp; ls"`
    {/}

Copy files from local to remote
:   `scp file file2 file3 ... user@the.server.edu:path/to/destination/`

    {.example ...} `scp testfile.c mst3k@portal.cs.virginia.edu:code/demo1/`
    {/}
    
    Note that `scp` will not create directories, but `ssh` can:
    
    {.example ...} `ssh mst3k@portal.cs.virginia.edu mkdir -p code/demo1/`
    {/}

Copy files from remote to local
:   `scp user@the.server.edu:path/to/source/filename path/to/destination/`
    
    (use `./` for "put it where I am")

    {.example ...} `scp mst3k@portal.cs.virginia.edu:code/demo1/testfile.c ./`
    {/}


