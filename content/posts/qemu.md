+++
date = 2020-07-24T06:00:00Z
lastmod = 2020-07-24T06:00:00Z
title = "QEMU"
subtitle = "QEMU Tips and Tricks"
+++

QEMU is an indispensable tool for the virtual machine inclined. It's a command
line utility for running virtual machines.

Main documentation: https://www.qemu.org/docs/master/system/

## Example Flags

Run a 64 bit Intel / AMD system

```console
qemu-system-x86_64 ...
```

### KVM

Without KVM your VM will be VERY VERY VERY slow. You'll want to enable this.

```
  -enable-kvm
```

Make sure your use account has access to `/dev/kvm`, use `chown` to make the
group `kvm`, and add your user to that group. You'll need log out and log back
in for changes to take effect. Or run `bash --login`.

```console
$ groupadd kvm
$ sudo usermod -aG kvm $USER
$ ll /dev/kvm
crw-rw-rw- 1 root root 10, 232 Jul 22 12:10 /dev/kvm
$ chown root:kvm /dev/kvm
$ ll /dev/kvm
crw-rw-rw- 1 root kvm 10, 232 Jul 22 12:10 /dev/kvm
```

### Multiple CPUs

```
  -smp cpus=4
```

More detailed

```
  -smp sockets=1,cpus=4,cores=2 -cpu host
```

### Memory

```
  -m 8192M
```

### Networking - NAT

```
  -netdev user,id=mynet0 \
  -device virtio-net-pci,netdev=mynet0
```

You can also do what's known as "bridged" networking. It can be a bit of a mess
though. It's covered in the main QEMU documentation at the top.

> TODO Cover bridged networking

### USB

You may want to give a VM control over a USB device, such as a USB NIC.

Creating a ehci device and attaching the usb device to it is important!

Use `lsusb -v` to find the idProduct (productid) and idVendor (vendorid)

```
  -usb \
  -device usb-ehci,id=ehci \
  -device usb-host,bus=ehci.0,vendorid=0x0424,productid=0xEC00
```

### Share small files

```
  -virtfs local,path=$PWD/share,mount_tag=host0,security_model=mapped-file,id=host0
```

### Fast Random Number Generator

```
  -device virtio-rng-pci
```

### Port Forwarding

```
  -net \
    user,hostfwd=tcp::2222-:22,hostfwd=tcp::4444-:2222
```

### Guest Image

For a raw `.img` or `.iso`

```
  -drive \
    file="image.iso",if=virtio,aio=threads,format=raw
```

For a `.qcow2`

```
  -drive \
    file="image.qcow2",if=virtio,aio=threads,format=qcow2
```

### Kernel

Boot directly to a Linux kernel binary (skips some BIOS stuff)

```
  -kernel \
    "linux-source-tree/arch/x86/boot/bzImage"
```

### Kernel cmdline

The `root*` options here correspond to the
[Host Filesystem Passthrough](#host-filesystem-passthrough) section.

```
  -append \
    "console=ttyS0 rootfstype=9p root=fsdev-root ro rootflags=trans=virtio,version=9p2000.u init=/usr/lib/systemd/systemd"
```

### Disable GUI

```
  -nographic
```

### CPU Emulation

Specify `host` to have QEMU not emulate another CPU, just use the host CPU.

```
  -cpu host
```

### Specify BIOS

You'll need this if you want to use UEFI

```
  -bios \
    "path/to/OVMF.fd"
```

### BIOS Debugging Connection

```
  -chardev \
    pipe,path=qemudebugpipe,id=seabios \
  -device \
    isa-debugcon,iobase=0x402,chardev=seabios
```

Reference: https://www.seabios.org/Debugging#Debugging_with_gdb_on_QEMU

### Host Filesystem Passthrough

Use a directory on the host as a filesystem for the guest.

This requires that the guest kernel has been configured with:

```
CONFIG_9P_FS=y
CONFIG_9P_FS_POSIX_ACL=y
CONFIG_9P_FS_SECURITY=y
CONFIG_NET_9P=y
CONFIG_NET_9P_VIRTIO=y
```

*9P fs is buggy as all hell* In particular, fsync seems to be broken.

```
  -fsdev \
    local,id=fsdev-root,path="${CHROOT}",security_model=passthrough,readonly \
  -device \
    virtio-9p-pci,fsdev=fsdev-root,mount_tag=/dev/root
```

Make sure you're using the [corresponding kernel cmdline](#kernel-cmdline)
options.

### Creating A Bootable UEFI Guest Image

Create `qcow2` image

```console
$ qemu-img create -f qcow2 image.qcow2 20G
```

> Source of NBD commands: https://gist.github.com/shamil/62935d9b456a6f9877b5

Enable network block devices

```console
$ sudo modprobe nbd max_part=8
```

Map the image file to the `/dev/nbd0` network block device

```console
$ sudo qemu-nbd --connect=/dev/nbd0 image.qcow2
```

Create GPT partition table (UEFI)

```console
$ sudo parted /dev/nbd0 << 'EOF'
mklabel gpt
mkpart primary fat32 1MiB 261MiB
set 1 esp on
mkpart primary linux-swap 261MiB 10491MiB
mkpart primary ext4 10491MiB 100%
EOF
```

Format partitions

```console
$ sudo mkfs.fat /dev/nbd0p1
$ sudo mkswap /dev/nbd0p2
$ sudo mkfs.ext4 /dev/nbd0p3
```

Unmount and disconnect

```console
$ sudo umount -R /mnt/somepoint/
$ sudo qemu-nbd --disconnect /dev/nbd0
```

> parted commands from: https://wiki.archlinux.org/index.php/Parted#UEFI/GPT_examples

### CPU Hotplug

```conole
  -smp 1,maxcpus=2 -qmp unix:/tmp/q,server,nowait
```

In another shell

```console
$ sudo qemu/scripts/qmp/qmp-shell -p -v /tmp/q
Welcome to the QMP low-level shell!
Connected to QEMU 4.1.0

(QEMU) device_add id=cpu-2 driver=host-x86_64-cpu socket-id=1 core-id=0 thread-id=0 die-id=0
```

Back in your Linux guest you'll see
`[   51.190460] CPU1 has been hot-added` in the dmesg logs.

To initialize the CPU within the guest

```console
# echo 1 > /sys/devices/system/cpu/cpu1/online
```

> Reference: https://wiki.qemu.org/Features/CPUHotplug

### Precompiled UEFI Firmware

https://cdn.download.clearlinux.org/image/OVMF.fd

```
UEFI_BIOS="-bios OVMF.fd"

if [ -f OVMF_VARS.fd -a -f OVMF_CODE.fd ]; then
    UEFI_BIOS=" -drive file=OVMF_CODE.fd,if=pflash,format=raw,unit=0,readonly=on "
    UEFI_BIOS+=" -drive file=OVMF_VARS.fd,if=pflash,format=raw,unit=1 "
fi
```
