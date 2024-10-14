+++
date = 2020-07-24T06:00:00Z
lastmod = 2020-10-06T08:00:00Z
title = "Linux Kernel Development Tips And Tricks"
subtitle = "Working on the Linux kernel itself"
+++

# Resources

- https://linux-kernel-labs.github.io/refs/heads/master/index.html

# Videos

- Kernel Recipes 2014 - The Linux Kernel, how fast it is developed and how we stay sane doing it
  - https://www.youtube.com/watch?v=DmDmXDtaz-U
- LPC2019 - Reflections on kernel quality, development process and testing
  - https://www.youtube.com/watch?v=iAfrrNdl2f4
- virtio
  - https://blogs.oracle.com/linux/post/introduction-to-virtio

# Quickstart

We are going to download the Git repositories of QEMU and the Linux kernel. We
are not going to cover what dependencies are required. You should read the
reference links for each section to help you get the compilers and other build
tools you will need.

## Building the Kernel

```bash
git clone https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
cd linux
git checkout v5.8
# Copy your system's config into the kernel source tree to use as your config
# You can modify it too if you want but you can be pretty sure this will give
# you a working kernel without having to edit it
cp $(ls /boot/config-* | head -n 1) .config
make olddefconfig
# Build just the bzImage needed to boot the VM. If you want to build all the
# modules too, just leave off the last argument (bzImage)
make -j $(($(nproc)*4)) bzImage
```

> References
>
> - [Fedora Build Dependencies](https://docs.fedoraproject.org/en-US/quick-docs/kernel/build-custom-kernel/#_get_the_dependencies)
> - [Ubuntu Build Dependencies](https://wiki.ubuntu.com/Kernel/BuildYourOwnKernel#Build_Environment)

## Building QEMU

```bash
git clone https://git.qemu.org/git/qemu.git
cd qemu
git checkout v5.1.0
git submodule init
git submodule update --recursive
mkdir build
cd build
../configure --target-list=x86_64-softmmu
make -j $(($(nproc)*4))
```

> References
>
> - https://wiki.qemu.org/Hosts/Linux
> - https://www.qemu.org/download/#source

## Running your VM

Put the following script in a file called `run-vm.sh`. It will be the way you
run your development Kernel in a virtual machine. You can modify it as you wish.

<script src="https://gist.github.com/pdxjohnny/a0dc3a58b4651dc3761bee65a198a80d.js"></script>

Here's how you download it, make it executable, and run it.

```bash
curl -o run-vm.sh -fSL https://gist.github.com/pdxjohnny/a0dc3a58b4651dc3761bee65a198a80d/raw/da2f456c9ecbb56bf84ee30c8c83a2762e86fb43/run-vm.sh && \
echo "eb5c49fb0aff6b3293ad6f4bd8c7a9c32df97f40d3c8c4fe404b72c1e9c283b44e714be493ce88b5f22e5bb717b8f71d  run-vm.sh" | sha384sum -c - && \
chmod 755 run-vm.sh
./run-vm.sh
```

## Tools

- [b4](https://pypi.org/project/b4/)

  - Grab patches from mailing list

## Exploitation

SMEP disable via ROP, KASLR bypasss via DMESG

- https://web.archive.org/web/20171029060939/http://www.blackbunny.io/linux-kernel-x86-64-bypass-smep-kaslr-kptr_restric/

## Hacking With VIM

https://stackoverflow.com/questions/33676829/vim-configuration-for-linux-kernel-development

## Build In Tree Modules

```bash
make -j $(($(nproc)*4)) M=arch/x86/kvm/ modules
```

## Install Kernel To chroot

Here's how you install a kernel to a chroot. Or tar it up and
send it over to another system for use there.

```bash
# Edit your config
make olddefconfig
export INSTALL_MOD_PATH=/path/to/your/chroot
export INSTALL_PATH=${INSTALL_MOD_PATH}/boot
mkdir -p "${INSTALL_PATH}"
# Use number of cores times 4, this usually is about how many
# threads your system can run
make -j $(($(nproc)*4))
make install
make modules_install -j $(($(nproc)*4))
```

## Chroot

Instead of doing a regular chroot use `systemd-nspawn` which gives you a more fully featured chroot.

https://wiki.archlinux.org/index.php/Systemd-nspawn

## Debugging With GDB

If you are lucky enough to be working on something that's in a VM you can get
GDB working!

You need to run the kernel without KASLR (on the cmdline that's `nokaslr`).

> If you don't disable KASLR gdb can't set breakpoints or display the source to you

Then you run QEMU with `-s -S`.

- `-s` says enable GDB on port `1234`
- `-S` says wait to start the VM until GDB connects.

Run GDB with `gdb ./vmlinux`

Now run

```console
(gdb) target remote 127.0.0.1:1234
```

To view the source code, type `layout src`

To focus back on the typey type window, type `focus cmd`


```c
   ┌──arch/x86/kernel/machine_kexec_64.c─────────────────────────────────────────────────────────────────┐
   │353                 if (result)                                                                      │
   │354                         return result;                                                           │
   │355                                                                                                  │
   │356                 /* update purgatory as needed */                                                 │
   │357                 result = arch_update_purgatory(image);                                           │
   │358                 if (result)                                                                      │
   │359                         return result;                                                           │
   │360                                                                                                  │
   │361                 return 0;                                                                        │
   │362         }                                                                                        │
   │363                                                                                                  │
   │364         void machine_kexec_cleanup(struct kimage *image)                                         │
   │365         {                                                                                        │
   │366                 free_transition_pgtable(image);                                                  │
   │367         }                                                                                        │
   │368                                                                                                  │
   │369         /*                                                                                       │
   │370          * Do not allocate memory (or fail in any way) in machine_kexec().                       │
   │371          * We are past the point of no return, committed to rebooting now.                       │
   │372          */                                                                                      │
   │373         void machine_kexec(struct kimage *image)                                                 │
B+>│374         {                                                                                        │
   │375                 unsigned long page_list[PAGES_NR];                                               │
   │376                 void *control_page;                                                              │
   │377                 int save_ftrace_enabled;                                                         │
   │378                                                                                                  │
   │379         #ifdef CONFIG_KEXEC_JUMP                                                                 │
   │380                 if (image->preserve_context)                                                     │
   │381                         save_processor_state();                                                  │
   │382         #endif                                                                                   │
   │383                                                                                                  │
   │384                 save_ftrace_enabled = __ftrace_enabled_save();                                   │
   │385                                                                                                  │
   │386                 /* Interrupts aren't acceptable while we reboot */                               │
   │387                 local_irq_disable();                                                             │
   │388                 hw_breakpoint_disable();                                                         │
   │389                                                                                                  │
   │390                 if (image->preserve_context) {                                                   │
   │391         #ifdef CONFIG_X86_IO_APIC                                                                │
   │392                         /*                                                                       │
   │393                          * We need to put APICs in legacy mode so that we can                    │
   │394                          * get timer interrupts in second kernel. kexec/kdump                    │
   │395                          * paths already have calls to restore_boot_irq_mode()                   │
   │396                          * in one form or other. kexec jump path also need one.                  │
   └─────────────────────────────────────────────────────────────────────────────────────────────────────┘
remote Thread 1.1 In: machine_kexec                                          L374  PC: 0xffffffff810635f0
(gdb) b *machine_kexec
Breakpoint 1 at 0xffffffff810635f0: file arch/x86/kernel/machine_kexec_64.c, line 374.
(gdb) c
Continuing.

Breakpoint 1, machine_kexec (image=0xffff888006f96800) at arch/x86/kernel/machine_kexec_64.c:374
(gdb)
```

Core dumps and kgdb: https://elinux.org/images/f/f0/Bingham.pdf

## KVM

https://lwn.net/Articles/658511/

### Nested and VMCS

https://web.archive.org/web/20191105205408/http://events19.linuxfoundation.org/wp-content/uploads/2017/12/Improving-KVM-x86-Nested-Virtualization-Liran-Alon-Oracle.pdf

## Contributions

In your patches, write all about what you're doing. The kernel doesn't
seem to like comments very much. Instead they rely on people writing
insanely detailed commit messages describing what they are changing.

Every time you change the kernel you're changing something that has been
working fine for someone for X amount of long time. As such, you need to
be convincing for why your change should be accepted! Make your coverletter
and commit messages very detailed. The kernel is a big place. It may have
been a long time since someone reading your patch series has looked at the
place you're working. Be courtious and remind them of all the moving pieces
involved in what you're doing, and why and how you're changing them.

### Etiquette

- You can say `patch` or `patchset` in your coverletter. Just not in the
  commit messages themselves.

## KVM

### VMX

When a VM enters VMX root mode (aka "I'm gonna run some VMs mode") using
the `VMXON` instruction, it sets up what's called a `VMCS` for each VM
/ virtualized processor it wants to run. All VMX related instructions
operate on whatever VMCS we pointed to via the `VMPTRLD` instruction.

### Nested

- L0 == The host

- L1 == The guest

- L2 == The guest of the guest

`vmx/nested.c` is responsible for emulating VMX for L1. When L1 does a
`VMLAUNCH` or `VMRESUME`, `enter_guest_mode` is called, which means that
`is_guest_mode` will now return `true`.

### Terminology

Root mode = L0 when not running a virtual machine

Non-root mode = L0 when running a virtual machine L(n)

## Assembly

What is `jmp 1f`?

https://stackoverflow.com/a/27353169/12310488
