---
title: File-based chat system
...

When you log into a department server
you have access to a large public shared space called `/bigtemp`.
I have placed in that directory a folder called `/bigtemp/coa1`
which we will use like a wall of mail cubbyholes for this assignment.

# Outline

You will write a program that will do the following:

1. If there is a file named `/bigtemp/coa1/mst3k.chat`, where `mst3k` is replaced by your ID,
    the program will show that file's contents to the user and then empty the file.

2. The program will ask the user the computing ID of the person they want to send a message to.

3. The program will ask the user what message to send them.

4. The program will append your ID, followed by a colon, followed by the message, to the file
    `/bigtemp/coa1/mst3k.chat`, where `mst3k` is replaced by the recipient's ID, and change its permissions so others can read and write it.

You are welcome to assume that no string will be bigger than 4096 characters and use a global `char buffer[4096]`
instead of trying to handle `malloc`, etc, in this lab.

You'll probably find it useful to work in pairs so you can send each other messages.

# Useful pieces

## Getting your ID

Linux provides access to several "environment variables".
One of them (`USER`) has, as its value, your current login ID.
You can retrieve this using the `getenv` function in `stdlib.h`:

{.example ...} The following will print your ID to the terminal,
duplicating the behavior of the built-in `whoami` command.

````c
int main() {
    char *me = getenv("USER");
    puts(me);
    return 0;
}
````
{/}

## Building a string

There are two basic tools for building a string out of component parts.
Either one will work fine for this task, and both
Both of them require a pre-allocated destination buffer.

### `strcat`

The `strcat` function from `string.h` is similar to `+=` for strings in Java or Python.

{.example ...} After running the following

````c
buffer[0] = '\0';
strcat(buffer, "/bigtemp/coa1/");
strcat(buffer, "mst3k");
strcat(buffer, ".chat");
````

the array `buffer` contains `"/bigtemp/coa1/mst3k.chat"`
{/}

### `sprintf`

The `snprintf` function from `stdio.h` does a formatted string construction, like the string's `.format` method in Java or Python.
It uses the same formatting specifiers as `printf`, with both the power and complexity that that entails.

{.example ...} After running the following

````c
snprintf(buffer, 4096, "/bigtemp/%s/%s.chat",
    "coa1",
    "mst3k",
);
````

the array `buffer` contains `"/bigtemp/coa1/mst3k.chat"`.
It will not overrun `4096` characters, even if it were given very large strings to format.
{/}

## Working with files

To work with files, you want to `fopen` them, access their contents, and then `fclose` them.
The following components will help:

Opening:

- `FILE *inbox = fopen(filepath, "r");`{.c} returns `NULL` if `filepath` is not the path to a real file, and a pointer to a `FILE` structure opened in read-only mode otherwise.

    There is a special `FILE *` named `stdin` that is always open and reads from what the user types into the terminal window.

- `FILE *outbox = fopen(filepath, "a");`{.c} returns `NULL` if `filepath` exists but you are not allowed to write to it, and a pointer to a `FILE` structure opened in append-only mode otherwise.

    There is a special `FILE *` named `stdout` that is always open and displays to the terminal window.

Writing:

- `fwrite(buffer, sizeof(char), 23, outbox);`{.c} writes 23 `char`-sized values from `buffer` to `FILE *outbox`{.c}.
    Use it if you know how many bytes you want to write.

- `fprintf(outbox, "%s: %d\n", "mst3k", 2501);`{.c} writes `mst3k 2501` and a newline to `FILE *outbox`{.c}.
    Like `snprintf`, this gives a lot of power with corresponding complexity.

Reading:

- `size_t got = fread(buffer, sizeof(char), 4096, inbox);`{.c} reads up to 4096 `char`-sized values from `FILE *inbox`{.c} into `buffer`, and returns the number read (which is often less than 4096 if `inbox` did not have that many characters).

    This does not work well for `stdin` as the `inbox` parameter, as you don't generally know how many characters the user will type.
    `fgets` is better for most user interactions.

- `char *line = fgets(buffer, 4096, inbox);`{.c} reads one line of text from `FILE *inbox`{.c}, or 4096 characters, whichever is less. It returns `buffer` on success and `NULL` on failure. The returned string includes the newline, as e.g. `"mst3k\n"`{.c}

    This works well for `stdin` as the `inbox` parameter, as users generally type one line at a time.

Permissions, etc:

- `chmod("/bigtemp/coa1/mst3k.chat", 0666);`{.c} sets the permissions for `mst3k.chat` based on a bit-vector of flags, specified in octal (hence the leading `0`):

    - The three octal digits are (in written order) the owner, group, and others permissions.
    - If re-written in binary (e.g. change `07` to 111~2~), the bits mean may-read, may-write, and may-execute, in order.
    - We want the files used in the chat to be writeable by any user, so `0666` is a reasonable permission set.

- `truncate("/bigtemp/coa1/mst3k.chat", 0);`{.c} truncates a file so its new size is 0 bytes.
    This is useful in re-setting a chat file after the user has read its contents.


# Testing tips

You can manually check for the existance and permissions of a file by running

````bash
ls -l /bigtemp/coa1/mst3k.chat
````

Note that many bugs end up creating the wrong file name; the following will list all files in the directory with the newest file last

````bash
ls -ltr /bigtemp/coa1/
````

You can force-add a message to a mailbox by running

````bash
echo "This is a message" >> /bigtemp/coa1/mst3k.chat
chmod 666 /bigtemp/coa1/mst3k.chat
````

You can read all messages by running

````bash
less /bigtemp/coa1/mst3k.chat
````

These can be useful in testing different aspects of your program independently.
But don't mess with other student's mailboxes in this way without their permission.

If you try to read from a non-existent file, open a file in a non-existent directory, or access a file for which you do not have permissions, the functions you use will fail (and set `errno`).

# Example run

The following shows how a pair of students, `aa1a` and `tj0uva` might chat with one another.
What the user types is shown <ins>like this</ins>.

+---------------------------------------+---------------------------------------+
| User `aa1a` does                      | User `tj0uva` does                    |
+:======================================+:======================================+
| <pre>                                 |                                       |
| $ <ins>./a.out</ins>                  |                                       |
| You have no new messages              |                                       |
| Who would you like to message?        |                                       |
| <ins>tj0uva</ins>                     |                                       |
| What do you want to say?              |                                       |
| <ins>Hello, is this working?</ins>    |                                       |
| </pre>                                |                                       |
+---------------------------------------+---------------------------------------+
|                                       | <pre>                                 |
|                                       | $ <ins>./a.out</ins>                  |
|                                       | Your messages:                        |
|                                       | aa1a: Hello, is this working?         |
|                                       | Who would you like to message?        |
|                                       | <ins>aa1a</ins>                       |
|                                       | What do you want to say?              |
|                                       | <ins>Yes, works well!</ins>           |
|                                       | </pre>                                |
+---------------------------------------+---------------------------------------+
|                                       | <pre>                                 |
|                                       | $ <ins>./a.out</ins>                  |
|                                       | You have no new messages              |
|                                       | Who would you like to message?        |
|                                       | <ins>aa1a</ins>                       |
|                                       | What do you want to say?              |
|                                       | <ins>How are you, by the way?</ins>   |
|                                       | </pre>                                |
+---------------------------------------+---------------------------------------+
| <pre>                                 |                                       |
| $ <ins>./a.out</ins>                  |                                       |
| Your messages:                        |                                       |
| tj0uva: Yes, works well!              |                                       |
| tj0uva: How are you, by the way?      |                                       |
| Who would you like to message?        |                                       |
| <ins></ins>                           |                                       |
| </pre>                                |                                       |
+---------------------------------------+---------------------------------------+

Some notes for the ideal implementation:

1. Use `getenv` to figure out the name of the user running the program. Don't hard-code it or ask them.
    - optionally, if `argc` is 2, use `argv[1]` instead so users can run as e.g. `./a.out tj0uva` to pretend to be TJ. If you do that, don't go erasing other student's mailboxes...
1. After showing the user their messages, truncate their mailbox
1. Append messages to the recipient's mailbox. Don't erase the file first.
1. `chmod` each file you touch to be `0666` so that others can work with it too.
1. Prepend the current user's ID and a `": "`{.c} to each message you put in a mailbox so the recipient can see who wrote it.
1. If the user does not type an ID when asked, don't ask for a message afterwards.

You don't need to do all of that to check off (showing the TA a basic implementation is enough),
but we encourage you to work on all of the above features to better prepare you for future assignments.
