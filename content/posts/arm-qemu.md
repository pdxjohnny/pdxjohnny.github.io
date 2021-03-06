+++
date = "2016-11-09T14:57:07-08:00"
title = "ARM workflow with qemu and arm-none-eabi"

+++

This is just the markdown portion of set of files which can be found here:
https://gist.github.com/pdxjohnny/3de9a9bdd38cacf3ea394207762f1002

This should get you up and running writing ARM assembly without hardware.

Clone this the repo for this turtorial.

```
git clone https://gist.github.com/pdxjohnny/3de9a9bdd38cacf3ea394207762f1002 arm-qemu
```

## Dependencies

The first step is to install the necessary packages. These are the
arm-none-eabi tool chain and qemu with arm support.

#### Arch Linux

```
sudo pacman -S arm-none-eabi-gcc arm-none-eabi-binutils arm-none-eabi-gdb \
  arm-none-eabi-newlib qemu qemu-arch-extra
```

#### Ubuntu

```
sudo apt -y install make qemu-system-arm \
    gcc-arm-none-eabi binutils-arm-none-eabi gdb-arm-none-eabi \
    libstdc++-arm-none-eabi-newlib libnewlib-arm-none-eabi
```

## GDB

In `.gdbinit` we have placed commands which gdb will run on startup. But to
make this work the `.gdbinit` file in our home directory needs to say its ok
for gdb to load this `.gdbinit` file. To do that we just add the directory to
the auto-load safe-path.

```
echo "set auto-load safe-path $PWD" >> ~/.gdbinit
```

## Building

The Makefile should have plenty of comments to help you understand what is
being done in it. It takes all the `.s` assembly files in the current directory
and compiles them into object files. Then it runs the linker to create the ELF
binary. All of this is done with arm-none-eabi-gcc rather than your regular
gcc for host programs.

```
make
```

Will rebuild all the modified `.s` files into their object file forms and
relink to the binary. Run `make clean all` if you are having really weird
errors. That usually fixes things.

## Running

To run you can do `qemu-arm ./main`. But hey why not put it in the Makefile
right.

```
make all qemu
```

Will rebuild any changed files and run the created binary in qemu.

## Debugging

Oh you ran the program and everything exploded? Time to debug.

```
make all gdb
```

Will rebuild all your source files and start the program in qemu with it as a
gdb target on port 1234, so make sure nothing else is using that port or change
it in the `.gdbinit` file and `Makefile`.

## Help nothing works

Comment on the gist with the problem so we can figure it out and everyone else
can see the solution.
