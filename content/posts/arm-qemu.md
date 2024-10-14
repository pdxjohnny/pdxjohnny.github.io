+++
date = "2016-11-09T14:57:07-08:00"
title = "ARM workflow with qemu and arm-none-eabi"

+++

This is just the markdown portion of set of files which can be found here:
https://gist.github.com/pdxjohnny/3de9a9bdd38cacf3ea394207762f1002

This should get you up and running writing ARM assembly without hardware.

Clone this the repo for this tutorial.

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

## Arduino

```console
curl -sfL https://github.com/arduino/arduino-cli/releases/download/0.19.2/arduino-cli_0.19.2_Linux_64bit.tar.gz | tar xvz
# Hangs because proxies aren't set yet
timeout 1s ./arduino-cli config init --overwrite
./arduino-cli config set network.proxy $https_proxy
./arduino-cli core update-index
./arduino-cli core install arduino:avr
./arduino-cli compile --fqbn arduino:avr:leonardo .
```

```python
"""


Compile QEMU Version 5.1.0 or newer. 5.1.0 is when AVR support was introduced.

.. code-block:: console

    $ wget https://download.qemu.org/qemu-6.1.0.tar.xz
    $ tar xvJf qemu-6.1.0.tar.xz
    $ cd qemu-6.1.0
    $ ./configure --target-list="avr-softmmu"
    $ make -j $(($(nproc)*4))

Change directory to this file's parent directory and run using unittest

.. code-block:: console

    $ cd python/cmd_msg_test/
    $ python -u -m unittest discover -v
    test_connect (test_cmd_msg.TestSerial) ... qemu-system-avr: -chardev socket,id=serial_port,path=/tmp/tmpuuq3oqvj/socket,server=on: info: QEMU waiting for connection on: disconnected:unix:/tmp/tmpuuq3oqvj/socket,server=on
    reading message from arduino
    b''
    b''
    qemu-system-avr: terminating on signal 2 from pid 90395 (python)
    ok

    ----------------------------------------------------------------------
    Ran 1 test in 4.601s

    OK
"""
import os
import sys
import signal
import pathlib
import tempfile
import unittest
import subprocess
import contextlib
import dataclasses

import main

# top level directory in this git repo is three levels up
REPO_ROOT = pathlib.Path(__file__).parents[2].resolve()


@contextlib.contextmanager
def start_qemu(bios):
    with tempfile.TemporaryDirectory() as tempdir:
        socket_path = pathlib.Path(tempdir, "socket")
        qemu_cmd = [
            "qemu-system-avr",
            "-mon",
            "chardev=none",
            "-chardev",
            f"null,id=none",
            "-serial",
            "chardev:serial_port",
            "-chardev",
            f"socket,id=serial_port,path={socket_path},server=on",
            "-nographic",
            "-machine",
            "arduino-uno",
            "-cpu",
            "avr6-avr-cpu",
            "-bios",
            str(bios),
        ]
        qemu_proc = subprocess.Popen(qemu_cmd, start_new_session=True)
        serial_port_path = pathlib.Path(tempdir, "ttyACM0")
        socat_cmd = [
            "socat",
            f"PTY,link={serial_port_path},rawer,wait-slave",
            f"UNIX:{socket_path}",
        ]
        socat_proc = subprocess.Popen(socat_cmd, start_new_session=True)
        try:
            while not serial_port_path.exists():
                pass
            yield str(serial_port_path)
        finally:
            # Kill the whole process group (for problematic processes like qemu)
            os.killpg(qemu_proc.pid, signal.SIGINT)
            os.killpg(socat_proc.pid, signal.SIGINT)
        qemu_proc.wait()
        socat_proc.wait()


class RunQEMU(unittest.TestCase):
    """
    Base class which will start QEMU to emulate an Arduino Uno machine using the
    BIOS (the .elf output of arduino-cli compile) provided.

    qemu-system-avr from QEMU Version 5.1.0 or newer is required.

    Starts a new virtual machine for each test_ function.
    """

    BIOS = REPO_ROOT.joinpath("build", "serial_cmd_test.ino.elf")

    def setUp(self):
        self.qemu = start_qemu(self.BIOS)
        # __enter__ is called at the beginning of a `with` block. __exit__ is
        # called at the end of a `with` block. By calling these functions
        # explicitly within setUp() and tearDown() we ensure a new VM is created
        # and destroyed each time.
        self.serial_port = self.qemu.__enter__()

    def tearDown(self):
        self.qemu.__exit__(None, None, None)
        del self.qemu


class TestSerial(RunQEMU, unittest.TestCase):
    def test_connect(self):
        os.environ["SERIAL_PORT"] = self.serial_port
        subprocess.check_call([sys.executable, "main.py"])
```
