+++
date = 2020-07-29T18:00:00Z
lastmod = 2020-07-27T18:00:00Z
title = "Terminal Quickstart"
subtitle = "UNIX shell cheat sheet"
+++

<center>
<img alt="Its dangerous to go alone, take this! $ _" src="/images/terminal-quickstart.gif" />
</center>

## References

Haven't looked at the content of these but they might help.

- https://www.youtube.com/watch?v=UuJuq5wubaU&list=PL78ppT-_wOmvlYSfyiLvkrsZTdQJ7A24L

## How to read examples

You're about to see a lot of examples similar to this one

```console
$ echo Hello World
Hello World
```

The first line in the above example shows

- The *prompt* (`$ `)

- The *command* (`echo`)

- The *arguments* (`Hello World`)

The second line and all the lines following it until you see another prompt
`$ ` show the output which was a result of running a command. In the above
example. The output was `Hello World` (the same as the arguments, this is the
purpose of the `echo` command).

## Command

The first word (Including underscores or hyphens) after the prompt `$ ` is the
command.

```console
$ ls
  ^^
   \--- Command
```

## Argument

Everything after the command are called arguments.

```console
$ ls   -l   mydirectory
  ^^   ^^   ^^^^^^^^^^^
   |    |    \--- Argument
   |    |
   |    \--- Argument
   |
   \--- Command
```

## Running a command

To run a command. Type the command and it's arguments into your terminal /
shell. Then hit enter.

In the terminal, there is a general philosophy that no news means good news.
Many commands will only output text to the terminal if that is a part of their
core functionality. Otherwise commands usually output logging information or
error information.

## String

The term *string* means a sequence of characters. Strings are usually found
or referred to within quotes. One can use the word string to talk about any
sequence of characters. Where *character* likely means something that can be
found in the [ASCII](https://man7.org/linux/man-pages/man7/ascii.7.html) table.

```
       Char          Char
       ----------------------
       SPACE         `
       !             a
       "             b
       #             c
       $             d
       %             e
       &             f
       '             g
       (             h
       )             i
       *             j
       +             k
       ,             l
       -             m
       .             n
       /             o
       0             p
       1             q
       2             r
       3             s
       4             t
       5             u
       6             v
       7             w
       8             x
       9             y
       :             z
       ;             {
       <             |
       =             }
       >             ~
```

For example you could call each of the following a string

- "Hello World"

- /home/username

- UNIX

## Navigation and paths

A directory is another word for a folder

When talking about where something is (a directory, a file, a command, etc.) we
use the word *path*.

The last part of a path is the something (the directory, the file, the command,
etc.)

Whenever we have a string that contains all the information we'd need to get
from where we are to wherever something is, we call it a path.

- `.` means the directory you're in

- `..` means the directory above the directory you're in

- `/` means the path *separator*

The file system works like a tree. We call something a *full path* or an
*absolute path* when it starts with `/`.

The directory a full path starts with is called the *root*. It's name is "the
root directory", because we can't give it a name, since it's denoted via the
path separator. Whenever you see `/` as the only character in a path, or as the
first character of a path, think to yourself, that's the root directory.

Where `/` is not the first or only character, it's used to show separation
between files and directories.

Examples

- `/`

  - The root directory

- `/a/b/c.txt`

  - Start at the root directory. Then go directory `a`. Within directory `a`
    there is a directory `b`. Go into directory `b`. Within directory `b` there
    is a file named `c.txt`. This is the file the path is referencing

- `./a.out`

  - `.` means the current directory. The separator is seen after it. Then we see
    `a.out` which is the file the path is referencing

- `../updog`

  - Look in the directory above the one we're currently in for a file named
    `updog`

- `../../updog`

  - Look in the directory above the directory above the one we're currently in
    for a file named `updog`

## Useful Commands

Here are some commands that will be useful for you to understand where you are,
how to move around, and what stuff is in the directory (aka folder) you're in.

### Where am I?

Use the `pwd` command to find out where you are on your system

In the below example, we run the `pwd` command and it tells us we're in our
`HOME` directory.

```console
$ pwd
/home/username
```

### What files are here?

The `ls` command prints the contents of the directory we are currently in

```console
$ ls
file1  file2
```

We can supply it with the name of a directory as an argument to have it list the
contents of that directory

```console
$ ls my-directory
file1  file2
```

If we give `-lAF` as an argument to `ls` it will show us all the hidden files as
a long list.

```console
$ ls -lAF
total 0
-rw-rw-r--. 1 username username 0 Jul 30 18:00 file1
-rw-rw-r--. 1 username username 0 Jul 30 18:00 file2
```

### How do I move around?

Pass the path as the only argument to the `cd` command to change where you
currently are, known as your *current working directory*.

```console
$ cd somewhere
```

Pass no arguments to be taken to your home directory

```console
$ cd
```

### How to I create new directories?

```console
$ mkdir -p my-new-directory
```

### Download files?

The curl or wget commands download files when provided with a URL

wget will save the file as it's filename

```console
$ wget 'https://example.com/file.tar.gz'
```

curl usually needs to be told to follow redirects using `-L` and then to have it
save the file as the filename you'll need `-O` (which if you put it with the
`-L` you only need one `-`).

```console
$ curl -LO 'https://example.com/file.tar.gz'
```

> Sometimes one is installed on your system and the other isn't. Try the other
> one if you're seeing something along the lines of "Command not found"

### Extract zip and tar files?

When you download software it usually in a `.tar` or `.zip` file.

You can extract `.tar` files using the tar command.

```console
$ tar -xvf file.tar.gz
```

You can extract `.zip` files using the unzip command.

```console
$ unzip file.zip
```

## Editor - VIM

vim (aka vi) is a common editor you'll find installed on a lot of UNIX systems.

To edit `filename.c` pass it as an argument to vim

```console
$ vim filename.c
```

Press

- `i` to start editing

  - Use the arrow keys to move around

- `ESC` to get out of editing mode

  - `:w` and then enter to save the file

  - `:q` and then enter to quit

You can learn how to use vim by going to https://vim-adventures.com/

## Keyboard Shortcuts

Some keys or key combinations that can speed you up.

### Tab

Whenever you hit the Tab key on your keyboard (above Caps Lock) your shell will
attempt to auto complete whatever you're typing. Hit Tab twice for a list of
autocomplete options. Keep typing and hitting tab and eventually when there is
only one match left it will fill the rest of the word in.

> TODO Add asciinema video of Tab complete in action

### Ctrl-R

Start typing to reverse search through your past commands. Hit enter to run.

> TODO Add asciinema video of Ctrl-R complete in action
