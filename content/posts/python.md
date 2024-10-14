+++
title = "Python"
subtitle = "Python programming development guide and gotchas"
date = "2020-10-12T14:50:57-07:00"
lastmod = "2020-10-12T14:50:57-07:00"
+++

> This post is in progress

## Debugging

- https://github.com/prompt-toolkit/ptpython
- https://rpyc.readthedocs.io/en/latest/

## Development Environment

First, update pip! `pip` is the way you'll install Python libraries and
programs that already exist. You'll likely need to install some existing things
using `pip` which you might `import` from your Python files for use in your
project.

We use the `--user` flag to tell pip to install just for our user. This
eliminates the need for `sudo`.

```console
$ python3 -m pip install --user --upgrade pip
Collecting pip
  Downloading https://files.pythonhosted.org/packages/cd/82/04e9aaf603fdbaecb4323b9e723f13c92c245f6ab2902195c53987848c78/pip-21.1.2-py3-none-any.whl (1.5MB)
    100% |████████████████████████████████| 1.6MB 862kB/s
Installing collected packages: pip
Successfully installed pip-21.1.2
```

Before you install any Python package, always update three packages. `pip`,
`setuptools`, and `wheel`. These packages are used in the download and
installation process of many packages. Installation of packages may fail due to
them requiring features of these three packages only present in their latest
versions. Which is why we always update them before installing other packages.

```console
$ python3 -m pip install --upgrade pip setuptools wheel
Defaulting to user installation because normal site-packages is not writeable
Requirement already satisfied: pip in ./.local/lib/python3.6/site-packages (21.1.2)
Requirement already satisfied: setuptools in /usr/lib/python3.6/site-packages (39.2.0)
Collecting setuptools
  Downloading setuptools-57.0.0-py3-none-any.whl (821 kB)
     |████████████████████████████████| 821 kB 381 kB/s
Collecting wheel
  Downloading wheel-0.36.2-py2.py3-none-any.whl (35 kB)
Installing collected packages: wheel, setuptools
Successfully installed setuptools-57.0.0 wheel-0.36.2
```

## Installing packages

You should create a new "virtual environment" each time you have a new
python project. Typically for every directory with a `requirements.txt` in it,
you would also have a `.venv` directory in it with the virtual environment
which you installed the packages into.

https://packaging.python.org/en/latest/guides/installing-using-pip-and-virtual-environments/#creating-a-virtual-environment

## Installing packages globally

If you want packages to be available for use without activating a virtual
environment, you can install them to your user site-packages.

With a recent version of pip installed, Python will install to your `--user`
location even if you forget to specify. Or you can pass `--user` to be
explicit about it.

```console
$ python3 -m pip install --user --upgrade dffml
```

## `async def` and `await`

`await` is a keyword which means "give me the return value *when it's ready*".
The caller is understands the return value may not be ready right away. The
caller knows that the callee may not have the return value right away since the
callee function was defined with `async def`. Using the `await` keyword tells
Python that the caller wishes to pause execution (stay at the same line) and
wait for the callee's return value to be ready. Pausing of execution is usually
due to (but not limited to) network events, such as sending or receiving of
data, or the completion of work done within another thread or process.

The power of using `async def` (and
[`asyncio`](https://docs.python.org/3.7/library/asyncio-task.html) in general)
is that since the execution of a caller can be paused, multiple `async def`
functions can be run at the same time, an `async def` function is also known as
a coroutine. When a coroutine reaches the point where it would be waiting on an
event, such as completion of work in another thread or process, or a network
receive, etc. Python pauses the execution of that coroutine, and checks if there
is another coroutine that was waiting for an event which completed during the
running of the previous coroutine. If it finds a coroutine that was waiting for
a now complete event, it runs that coroutine until it either completes, or until
it is stuck waiting on another event.

## Drop to debugger on exception

**a.py**

```python
def func(a):
    if "a" in a:
        return True
    return False

def main():
    func(3)

if __name__ == "__main__":
    main()
```

If you pass `-i` you dump to a *fresh* interpreter session

```console
$ python -i a.py
Traceback (most recent call last):
  File "a.py", line 10, in <module>
    main()
  File "a.py", line 7, in main
    func(3)
  File "a.py", line 2, in func
    if "a" in a:
TypeError: argument of type 'int' is not iterable
>>> a
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'a' is not defined
```

If you run with pdb you get an interpreter session within
the same context.

- https://docs.python.org/3/library/pdb.html
- https://docs.pytest.org/en/7.0.x/how-to/failures.html#using-python-library-pdb-with-pytest

```console
$ python -m pdb a.py
> /tmp/tmp.aHjcycHqeF/a.py(1)<module>()
-> def func(a):
(Pdb) c
Traceback (most recent call last):
  File "/usr/lib/python3.7/pdb.py", line 1699, in main
    pdb._runscript(mainpyfile)
  File "/usr/lib/python3.7/pdb.py", line 1568, in _runscript
    self.run(statement)
  File "/usr/lib/python3.7/bdb.py", line 578, in run
    exec(cmd, globals, locals)
  File "<string>", line 1, in <module>
  File "/tmp/tmp.aHjcycHqeF/a.py", line 1, in <module>
    def func(a):
  File "/tmp/tmp.aHjcycHqeF/a.py", line 7, in main
    func(3)
  File "/tmp/tmp.aHjcycHqeF/a.py", line 2, in func
    if "a" in a:
TypeError: argument of type 'int' is not iterable
Uncaught exception. Entering post mortem debugging
Running 'cont' or 'step' will restart the program
> /tmp/tmp.aHjcycHqeF/a.py(2)func()
-> if "a" in a:
(Pdb) a
a = 3
(Pdb)
```

# `requirements.txt`

https://www.python.org/dev/peps/pep-0508/
