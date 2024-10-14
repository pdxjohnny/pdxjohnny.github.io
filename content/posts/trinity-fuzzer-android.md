+++
date = "2016-10-17T12:51:22-07:00"
title = "Trinity fuzzer on Android (on Intel)"

+++

## Step 1 - Clone trinity

Grab the source for trinity. We need to make some minor modifications.

> At the time of writing this the latest commit on master was
> 3a0e33d1db3214503316840ecfb90075d60ab3be adapt instructions as necessary. The
> basic idea of static linking and disabling of feature's you don't need is
> still the same.

```
git clone https://github.com/kernelslacker/trinity
cd trinity
./configure
```

## Step 2 - 32 bit and static compilation

Edit the make file and add `-m32` to any line containing `CFLAGS` and add
`-static` to any line containing `LDFLAGS`. Just one line not all of them that
say LD/CFLAGS. For example.

```
CFLAGS += -Wall -Wextra -g -O2 -I. -Iinclude/ -Wimplicit -D_FORTIFY_SOURCE=2 -D_GNU_SOURCE -D__linux__
```

Becomes

```
CFLAGS += -Wall -Wextra -g -O2 -I. -Iinclude/ -Wimplicit -D_FORTIFY_SOURCE=2 -D_GNU_SOURCE -D__linux__ -m32
```

And

```
LDFLAGS += -rdynamic
```

Becomes

```
LDFLAGS += -rdynamic -static
```

Just change two lines and you're done.

## Step 3 - Fix syscalls/send.c

I found that gcc 6.2.1 said that this is an error so it wouldn't let me
compile without this typecast. I haven't noticed anything strange by doing this
so I assume everything still works.

Change the line that reads

```c
proto->gen_packet(&si->triplet, ptr, &rec->a3);
```

To

```c
proto->gen_packet(&si->triplet, ptr, (size_t *) &rec->a3);
```

And you should now make it past that compilation error.

> syscalls/send.c:33:41: error: passing argument 3 of ‘proto->gen_packet’ from
> incompatible pointer type [-Werror=incompatible-pointer-types]

## Step 4 - Edit config.h

Take out anything you don't need or can't compile from `config.h`. For instance
Android recommends building on Ubuntu 16.04 so we were on a Ubuntu 16.04.1 LTS
machine which at the time of writing is Linux 4.4ish. So we couldn't compile
`fds/bpf.c` because the kernel headers didn't contain the right version of
`linux/bpf.h` (bpf_addr was missing some members).

The solution is to comment out `USE_BPF` from `config.h`.


```c
#define USE_BPF 1
```

After

```c
// #define USE_BPF 1
```

## Step 5 - Verify

Before we push to the device make sure it has a chance of working. This means
that it will be 32-bit statically linked. For the edits we just made to the
commit referenced at the top of this doc file reports the following.

```
file trinity
trinity: ELF 32-bit LSB executable, Intel 80386, version 1 (GNU/Linux), statically linked, for GNU/Linux 2.6.32, BuildID[sha1]=e533cc2db1db19c044bdbfa566c72299df7eefc2, not stripped
```

## Step 6 - Push and Run

Put in on the device and run from `/sdcard` folder.

```
adb root
adb remount rw
adb disable-verity
adb reboot
adb push trinity /data/trinity
adb shell 'cd /sdcard && /data/trinity --dangerous -V /dev/ 2>&1' | tee trinity-dev-1.log
```

Take the blue pill and profit
