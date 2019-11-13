---
title: Socket Chat
...

In this assignment, you'll take the server code we supplied and the client code you wrote in [lab 9](lab09-sockets.html) and combine them to make a command-line 2-party chat program.

# Requirements

Write a single file called `schat.c`. Use only C library functions and code you wrote in it: no third-party libraries or C++.

`schat.c` should have a `main` function that accepts command-line arguments.
If given no arguments, it should run in server mode; if given two arguments (the first an IP address, the second a port number) it should run in client mode, connecting to that server.

The server should begin by

1. Pick a random ephemeral-range port, like the lab server code did
1. Display that port number to the terminal, like the lab server code did
1. Listen for IPv4 connections on any IP address, like the lab server code did
1. `accept` only once, unlike the lab server's `accept` loop
1. `close` the `listen`er socket before working with the `accept`ed socket

The client should begin by

1. `connect`ing to the given IP address and port, like your lab client did

Both should then proceed as follows:

1. Repeatedly (i.e., in an infinite loop) use `poll`^[see `man 3 poll`{.bash} which has some example code in it. There's also a `man 2 poll`; you want `man 3 poll` instead] to pick either the connected socket (from the server's `accept` or the client's `connect`) or the standard input stream^[which is always already `open`ed as file descriptor `0`] to `read`^[Note: this means you'll need to use `POLLIN` as the `fds[i].events` instead of the `POLLOUT | POLLWRBAND` used in the manual page example.] from. Use a 1-minute^[60 thousand milliseconds] timeout for `poll`.
1. If `poll` returns a positive number (i.e., it succeeded),
    1. If the standard input has `revents` including `POLLIN`, `read` from standard input and `write` what you read to the socket.
    1. If the socket has `revents` including `POLLIN`, `read` from the socket and `write` what you read to standard output^[which is always already `open`ed as file descriptor `1`]. You may assume no single message is more than 4096 bytes, to avoid needing to loop your `read`/`write` calls.

# Undefined behaviors

This assignment does not require you to handle the following in any particular way:

1. Giving notice of server IP address
1. Failure of any of the socket, read, or write instructions
1. Handling the closing of the remote connection
1. Stopping if the `poll` call times out
1. Having any way to end the program other than Ctrl+C
1. Avoiding memory leaks
1. Clean display when the remote message arrives while the user it typing
1. Displaying who sent which message

You *do* need to avoid buffer overflows, use-after-free, and other memory bugs.
You are also *invited* to add sane behaviors for all of the above cases, but are not *required* to do so.

# Example

In a simple implementation, once the client and server connect everything that is typed in one will appear in the other after you press enter.

<table width="100%" style="height:25em;"><thead><tr><th width="50%">Server</th><th width="50%">Client</th></thead><tbody><tr><td style="vertical-align:top"><pre id="server" style="color:white;background:black;border:1ex solid black;">$ </pre></td><td style="vertical-align:top"><pre id="client" style="color:white;background:black;border:1ex solid black;">$ </pre></td></tr></tbody><table>
<script>
let events=[
'st./a.out\n',
'spportal04.cs.virginia.edu has address 128.143.69.114\nListening on port 55718\n',
'ct./a.out 128.143.69.114 55718\n',
'ctHi!\n',
'spHi!\n',
'stHello\n',
'cpHello\n',
'stHow are you today?\n',
'cpHow are you today?\n',
'ctFine, thanks; and you?\n',
'spFine, thanks; and you?\n',
'stI\'m good.\n',
'cpI\.m good.\n',
'ctOK. Bye.\n',
'spOK. Bye.\n',
'ct^C\n',
'cp$ ',
'stWait, don\'t go!\n',
'stHello?\n',
'st^C\n',
'sp$ ',
];
var i = 0;
var ai = 2;
function act() {
if (i == events.length) {
document.getElementById('client').innerHTML = '$ █';
document.getElementById('server').innerHTML = '$ █';
i = 0;
ai = 2;
setTimeout(act, 1000);
} else {
let row = events[i];
let dest = document.getElementById(row[0] == 'c'? 'client':'server');
if (row[1] == 'p') {
dest.innerHTML = dest.innerHTML.substr(0,dest.innerHTML.length-1) + row.substr(2)+'█';
i += 1
ai = 2;
setTimeout(act, 500);
} else {
dest.innerHTML = dest.innerHTML.substr(0,dest.innerHTML.length-1) + row[ai]+'█';
ai += 1;
if (ai < row.length) setTimeout(act, 100);
else { i += 1; ai = 2; setTimeout(act, 250); }
}
}
}
act();
</script>

As a reminder, you don't need to be exactly like the above; the initial display, format of output, and behavior of the server after the client ends are all [undefined](#undefined-behaviors) by this assignment.

# Tips

## This assignment requires looking things up

You will notice all we said about `poll` was "check the manual page".
This is by design. We hope you now know enough to learn a bit on your own and use it.

It is also possible to look things up online. Except for [online `man`-pages](http://man7.org/linux/man-pages/man3/poll.3p.html)^[And even those vary a lot in how up-to-date they are...], this is a bad idea.
In general, online example code is explaining something nuanced within a specific context,
and it takes some experience before you are able to tease out the part you can use from the context it is discussed within.
I expect if you go to a search engine to find example code, you'll end up with a lot of code that does not work or meet the required specification in some nuanced way.

## Testing on a non-server

The CS department servers (notably including servers `portal` redirects you to) all have *static IP Addresses* meaning tools like `host` can learn who they are. Your laptop probably does not.

If your laptop has a C compiler and `poll`, `socket`, etc., implemented, you can still test locally but you'll need to use the IP address `127.0.0.1` which always means "talk to myself", running the server and client in two different terminal windows at the same time.

## Use `lldb`

When you get a segfault, you should

1. recompile with `clang -g schat.c`
2. load in a debugger with `lldb a.out`
3. `run` and see the line of code that segfaulted

This will be far easier than trying to locate the segfault without a debugger.

You may also find a debugger useful for other kinds of errors.

## Feedback

We do not currently have plans to offer automated feedback on your code when you submit. We're also not planning on testing strange cases: if you can carry on a chat conversation with someone else, where either one of you can type several things in a row and it works, you should be good to go.

