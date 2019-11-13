---
title: Sockets lab
...

# Necessary background

As discussed in class `fcntl.h` and `unistd.h` provide functional wrappers around the internal operating-system abstraction of a **file-like object**: things the OS lets you read to and write from.
Conceptually the operating system maintains, for each running process, and **array** of these objects
and user code can interact with them by passing in **indexes** into this array, called "**file descriptors**."

In addition to actual files and the terminal, file descriptors can also be used to represent other communications channels.
Among those are a set of related concepts collectively called "sockets."

Creating a socket creates an object on the OS's file-like-objects array, but does not finish hooking it up.
For the TCP/IP sockets this lab will use, we'll need several other steps to do this.
In the end, we'll have two programs running at once, possibly on different computers, each with a socket
connected by a virtual two-way communication channel.

Each connected socket has exactly two ends: the local end you use in your code, and the remote end somewhere else.

A basic TCP/IP socket application uses three socket pairs:

- A server listening socket that connects a computer to the Internet and waits around for other computers to contact it
    - The remote end of this socket is held by the OS, which sends "I got a new connection attempt" messages to your code through it
- A client socket that contacts the server listening socket
    - The remote end of this socket is the server communication socket
- A server communication socket that the OS creates and sends through the server listening socket to your code
    - The remote end of this socket is the client socket

In typical "use words loosely" fashion, the client socket, server communication socket, and the connection between them is together often called simply a "socket".

# Address and Port

A socket connection requires an *address* which identifies which computer to contact and a *port* which helps the OS know which process the connection should be sent to.

## Port

Ports are [partially specified by IANA](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers).
We'll use a random port from the ephemeral port region.

The *server* will pick one using a random number generator seeded with the OS's number of the currently running process, which should minimize the risk of us picking one that another process is already using and means if we do we can re-run the program to get a new one:

````c
srandom(getpid()); // random seed based on this process's OS-assigned ID
int port = 0xc000 | (random()&0x3fff); // random element of 49152–65535
````

The *client* will need to know the port that the server it wants to contact is listening on,
so we'll have the port be a command-line parameter in the client.

## Address

TCP/IP addresses tell us what computer and program we're talking to. They are somewhat involved to explain (we'll go into more on these in COA2),
but are stored in a `struct sockaddr_in` declared in `<netinet/in.h>`.
For our uses, we'll need to (a) create one of these, (b) zero it out, and then (c) set three fields:

- `ipaddr.sin_family = AF_INET;`{.c} says we are using an IPv4 address, still the most widely-supported address family though it is starting to be replaced by IPv6.

- `ipOfServer.sin_port = htons(port);` puts the [Port] number into the address structure.
    The `htons` is an endian-changing function; because computers of both endiannesses can attach to the Internet, network communications are handled "network byte order" (i.e., big endian), requiring conversion functions like `htons` and `htonl`.

- `ipOfServer.sin_addr.s_addr` needs to be `htonl(INADDR_ANY)` for the server to say it is listening for communication from any other computer; for the client it instead needs to be `inet_addr(ip_address_of_server);` where `ip_address_of_server` will be a string containing four numbers separated by periods, like `"128.143.67.241"`{.c}.
    - You can learn the IP address of a URL by using the `host` command line tool.
    
        ````bash
        $ host portal01.cs.virginia.edu
        portal01.cs.virginia.edu has address 128.143.69.111
        ````
        
        There many be several other addresses listed; you want the one with four integers separated by periods.

# Important functions

The following are the main socket functions you need, in the order you'll need to use them:

+---------------------------------------+---------------------------------------+
| Server                                | Client                                |
+=======================================+=======================================+
|`socket(AF_INET, SOCK_STREAM, 0)`{.c}  |`socket(AF_INET, SOCK_STREAM, 0)`{.c}  |
|creates an unbound TCP/IP socket and   |creates an unbound TCP/IP socket and   |
|returns it's file descriptor.          |returns it's file descriptor.          |
+---------------------------------------+---------------------------------------+
|`bind(s, &ip , sizeof(ip))`{.c}        |                                       |
|asks the OS to reserve this port and   |                                       |
|address for socket `s`.                |                                       |
+---------------------------------------+---------------------------------------+
|`listen(s, 20)`{.c}                    |                                       |
|asks the OS to allow incoming          |                                       |
|connection attempts on socket `s`,     |                                       |
|hinting that we'd like to queue up as  |                                       |
|many as 20 attempts at once.           |                                       |
+---------------------------------------+---------------------------------------+
|`int c = accept(s, 0, 0)`{.c}          |`connect(s, &ip, sizeof(ip))`{.c}      |
|suspends your program until there is a |connects socket `s` to the server      |
|connection attempt on `s` and then     |identified in `ip`.                    |
|returns the server communication socket|                                       |
|as `c`.                                |                                       |
|                                       |                                       |
+---------------------------------------+---------------------------------------+
|`read` and `write` with `c` as needed  |`read` and `write` with `s` as needed  |
|to communicate with the client.        |to communicate with the server.        |
+---------------------------------------+---------------------------------------+
|`close(c)`{.c} to end communication.   |`close(s)`{.c} to end communication.   |
+---------------------------------------+---------------------------------------+
|either `accept` another connection or  |                                       |
|`close(s)`{.c} to stop listening.      |                                       |
+---------------------------------------+---------------------------------------+


# Your task

Below is the completely code of a server program that sends the same message to every client that connects:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>

const char *msg = "Congratulations, you've successfully received a message from the server!\n";

int main() {
    // start by getting a random port from the ephemeral port range
    srandom(getpid()); // random seed based on this process's OS-assigned ID
    int port = 0xc000 | (random()&0x3fff); // random element of 49152–65535

    // create an address structure: IPv4 protocol, anny IP address, on given port
    // note: htonl and htons are endian converters, essential for Internet communication
    struct sockaddr_in ipOfServer;
    memset(&ipOfServer, 0, sizeof(struct sockaddr_in));
    ipOfServer.sin_family = AF_INET;
    ipOfServer.sin_addr.s_addr = htonl(INADDR_ANY);
    ipOfServer.sin_port = htons(port);

    // we'll have one socket that waits for other sockets to connect to it
    // those other sockets will be the ones we used to communicate
    int listener = socket(AF_INET, SOCK_STREAM, 0);

    // and we need to tell the OS that this socket will use the address created for it
    bind(listener, (struct sockaddr*)&ipOfServer , sizeof(ipOfServer));

    // wait for connections; if too many at once, suggest the OS queue up 20
    listen(listener , 20);

    system("host $HOSTNAME"); // display all this computer's IP addresses
    printf("The server is now listening on port %d\n", port); // and listening port

    for(;;) {
        // get a connection socket (this call will wait for one to connect)
        int connection = accept(listener, (struct sockaddr*)NULL, NULL);
        if (random()%2) { // half the time
            write(connection, msg, 40); // send half a message
            usleep(100000); // pause for 1/10 of a second
            write(connection, msg+40, strlen(msg+40)); // send the other half
        } else {
            write(connection, msg, strlen(msg)); // send a full message
        }
        close(connection); // and disconnect
    }

    // unreachable code, but still have polite code as good practice
    close(listener);
    return 0;
}
```

Your job is 

1. Read enough `man`-pages that you understand what every line of the above code does.

and then to write a client program that

2. Accepts an IP address and a port number from the command line.
3. Connects to that server.
4. `read`s a message from the server and displays it on the command line.

Ideally your program should

- Verify that the connection worked, giving a reasonable error message if the IP and port combination failed to connect.
- Use proper `while`-loop structure to `read` all the data sent, even if not sent all at once (run the client repeatedly to test this).
- Also check the return values of every other function that returns error status (see the **return value** section in each function's manual page).
- `close` everything it `open`s and `free` everything it `malloc`s

# Testing your code

You'll need a server running at some known IP address and port.
You'll also need to run your client.

If you want to run your own server (alternatively, you can use a server another student is running if you wish),
you'll need two programs to run at once, meaning you need them to have different names, not just the default `a.out`.
The `-o` compiler flag helps with this; `clang mycode.c -o quidnunc`{.sh} names the resulting binary `quidnunc` instead of `a.out`, to be run as `./quidnunc`.

You'll need to leave the server running as long as you want to run your client.
Do this by opening two terminal windows and running the client in one, the server in the other.

**Make sure** you **kill** the server program when you are done working with it (e.g., by pressing Ctrl+C in that terminal window).
If too many people leave too many servers running, the portal back-end servers may eventually start running out of open ports and need to be restarted by the systems staff...


