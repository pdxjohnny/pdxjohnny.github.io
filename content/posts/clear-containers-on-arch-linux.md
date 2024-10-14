+++
date = "2017-01-17T21:06:04-08:00"
description = ""
title = "Clear Containers on Arch Linux"

+++

So you want the security of a virtual machine but the ease of use of docker and
containers? Well Clear Containers is the solution for you my friend. Here I am
going to show you how to install and configure the Clear Containers runtime on
your Arch Linux host.

I have created to packages in AUR to assist you in building the packages you
need.

### qemu-lite package

[qemu-lite][1] provides virtualisation for Clear Containers. It's a fork of qemu
that has a new machine type, pc-lite. Which is a traditional x86_64 machine
with a lot of things striped out. This lets them turn off a lot of things in
the Linux kernel config and thus increases the speed at which the VM (Container
in this case) is run. This also decreases time it takes the virtualised
container to start up.

Lets build qemu-lite, which will replace the hosts qemu package. We just need
to clone the package from the AUR, build it with makepkg, and install it with
pacman.

pacman will complain about qemu-arch-extra and qemu-launcher needing the
regular qemu so if you have those installed you need to remove them. They will
not work with qemu-lite.

```
# Download the one I built for you
wget https://github.com/pdxjohnny/pdxjohnny.github.io/releases/download/CCARCH/qemu-lite-2.7.1-1-x86_64.pkg.tar.xz
# Or build it yourself!
git clone https://aur.archlinux.org/qemu-lite.git
cd qemu-lite
makepkg -cs
# Then install
sudo pacman -U qemu-lite-2.7.1-1-x86_64.pkg.tar.xz
```

### cc-oci-runtime package

The [Clear Containers runtime][2] is an alternative to runc. You can have some
containers running runc and some running cor (Clear Containers runtime), or all
one or the other. It provides the management of qemu-lite so you can use these
VMs as if they were containers. Basically its where the magic happens.

```
# Download the one I built for you
wget https://github.com/pdxjohnny/pdxjohnny.github.io/releases/download/CCARCH/cc-oci-runtime-2.1.0-1-x86_64.pkg.tar.xz
# Or build it yourself!
git clone https://aur.archlinux.org/cc-oci-runtime.git
cd qemu-lite
makepkg -cs
# Then install
sudo pacman -U cc-oci-runtime-2.1.0-1-x86_64.pkg.tar.xz
```

### Configuration

Now we need to change how the docker daemon is run so that it knows about the
runtime he have just installed.


[1]: https://github.com/01org/qemu-lite
[2]: https://github.com/01org/cc-oci-runtime
