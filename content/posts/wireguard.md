+++
date = 2020-07-24T06:00:00Z
lastmod = 2020-07-24T06:00:00Z
title = "Wireguard HowTo"
subtitle = "How to use Linux's built in peer to peer VPN"
+++

Wireguard is a VPN that comes built into Linux kernels >= 5.6
It also has clients for OSs like Windows, OSX, and Android.

If you're looking for how to have a virtual LAN party, Wireguard
is a great way to do it. Since it's cross platform. You can even
play Windows games on Linux using Lutris and network them with
Wireguard to get Windows and Linux machines playing together.

## Checking If You Have It

To check if you're Linux distro already has wireguard support,
look in the modules directory.

> My version at the moment is `5.6.4-152.current`, yours will
> likely be different

```console
$ find /lib/modules/$(uname -r) -type f -name '*wireguard*.ko*'
/lib/modules/5.6.4-152.current/kernel/drivers/net/wireguard/wireguard.ko
```

If that doesn't show anything then check if it's built into the
kernel. This command will show some output if it is. On my system
it's a `.ko`, therefore it's not builtin, therefore this command
showed nothing.

```console
$ grep wireguard /lib/modules/$(uname -r)/modules.builtin
```

You'll also need the `wg` command line utility. Just type `wg` and
if you see command not found then you'll want to head over to
https://www.wireguard.com/install/ for details on how to install it.

## Getting It

If you don't have a version of the Linux kernel with wireguard
built in, you can compile and use the
[wireguard-go](https://git.zx2c4.com/wireguard-go/about/) project.
It won't be as fast as compiling the kernel module but it might be
a hell of a lot less steps.

If you don't want to take the performance hit, you can compile the
kernel module as seen in the
[compilation](https://www.wireguard.com/compilation/) instructions.

You'll build it, then add it to the kernel with `insmod`.

```console
$ sudo insmod wireguard-linux-compat/src/wireguard.ko
```

Check `dmesg`, if you see complaints about missing symbols,
you can recompile those modules that are missing, as seen
under kernel requirements. You'll want to find out how your
Linux distribution recommends compiling the kernel. This is
because many of them have patches that your system won't
work without added to the kernel. You might be able to run
the kernel from kernel.org, but don't count on that.

## Configuring It

The following is an example config file one might use.
There are comments within it to explain everything.

```ini
# The Interface section is for defining things about this
# machine
[Interface]
# The port wireguard will listen on for others to use as
# and Endpoint (along with this machines IP)
ListenPort = 60200
# On Windows when you click Add Tunnel it will generate
# a private key for you. On Linux you'll want to put the
# output of `wg genkey` here.
# The config files of the other machines in the network
# will need the public key that correseponds to this private
# key. On Windows it will show you the public key in a little
# box under the config file name when it's being edited. On
# Linux, you can run the following command to parse this file
# to get the private key and convert it into the public key
# $ grep 'PrivateKey = ' wg.conf | tail -n 1 | sed -e 's/.* = //g' | wg pubkey
PrivateKey = REPLACE_WITH_PRIVATE_KEY
# On Windows you'll want to specify the address for the
# machine like by removing the # in front. The setup.sh
# script that follows parses it out of this config file
# and uses it as your address. For Linux the wg tool will
# tell you there's an error if you uncomment it.
# You'll want to make sure you leave this as /24 and
# the other addresses as /32. See the References section
# for more details on this.
# Address = 192.168.4.4/24

# Each time you have a new compuer you want to connect to
# it'll need it's own [Peer] section.
[Peer]
# You'll want to specify the public key of the machine
# you want to connect to. The person with the config file
# of the machine you're trying to connect to needs to run
# the command with `wg pubkey` in the comment above PrivateKey
# to get their public key. They then should share it with you
# over some secure medium of communication, ideally end to
# end encrypted. If it gets tampered with, your wireguard
# connection is no longer secure.
PublicKey = 4QEX7I58pR5PaZNmDI2wmnsT/HvvFBkNc5wZJ00scXw==
# You'll want to put the IP that the other machine will be
# accessible at here. This is the value they have as Address
# under the [Interface] section of their wg.conf
AllowedIPs = 192.168.4.115/32
# If the person has a server they're running wireguard on and
# has exposed their ListenPort then that goes here. If another
# person has used their home router and port forwarded then
# that also can be used. The IP address here is the IP address
# of the machine as it is on the publicly addressable Internet,
# not your friends IP on their home network. This can be found by
# typing into Google, what's my IP, or using canihazip.com
Endpoint = 76.115.24.198:43022
# This is used to make sure the connection stays alive.
# It says to send a heartbeat / ping to the Endpoint every
# 25 seconds
PersistentKeepalive = 25

# Keep adding peers as you wish. At a minimum you'll need to
# add their public key and their address
[Peer]
PublicKey = 3StsslOTQlqMnd42UaKi9FdNu9GSTLi1WCaqwg8lkhc=
AllowedIPs = 192.168.4.3/32
```

If we wanted to setup a network where all machines would have
192.168.4.XXX addresses, we could do it as follows.

First, copy the example config file into a file named `wg.conf`.

You'll want to generate a new private key and add it to the file.

```console
$ sed -i "s#REPLACE_WITH_PRIVATE_KEY#$(wg genkey)#" wg.conf
```

You need to get your public key from your private key.
The following command parses the private key out of the
config file and generates the corresponding public key.

```console
$ grep 'PrivateKey = ' wg.conf | tail -n 1 | sed -e 's/.* = //g' | wg pubkey
rR7O5IvIq16RpntAqZCNmTd42nsI6Mq5139XMYwZ5hQ=
```

The output of this command is the public key which will
be used in one of the `[Peer]` sections in the config of
the other machines in your new wireguard network. It will
change whenever you regenerate the private key and replace
it in the config file.

You should communicate with the other people you're trying
to set up your new wireguard network with to make sure you
all choose a different IP address for the line that has
`Address = `. On Windows you'll need to uncomment that line.
In the [Using It](#using-it) section there will be a `setup.sh`
script, On Linux the script will parse the comment and use it
as your IP address within the wireguard network, do not
uncomment that line on Linux.

You can parse your config file to get your chosen address
using the following command.

```console
$ cat "${conf}"| grep 'Address = ' | sed -e 's/.* = //g' -e 's/\/24//g'
192.168.4.4
```

In this case, our machine will appear in the config file
of the other machines as follows

```ini
[Peer]
PublicKey = rR7O5IvIq16RpntAqZCNmTd42nsI6Mq5139XMYwZ5hQ=
AllowedIPs = 192.168.4.4/32
```

If you have the `ListenPort` exposed to the public internet,
via port forwarding or via firewall rules (or both). Then the
peer section for your machine in the config files of the other
people in your network will include the `Endpoint` and optionally
the `PersistentKeepalive` properties.

You can find your IP by running

```console
$ curl -s -w '\n' https://canihazip.com/s
24.21.101.158
```

The corresponding `[Peer]` section in other peoples configs
would then be the following.

```ini
[Peer]
PublicKey = rR7O5IvIq16RpntAqZCNmTd42nsI6Mq5139XMYwZ5hQ=
AllowedIPs = 192.168.4.4/32
Endpoint = 24.21.101.158:60200
PersistentKeepalive = 25
```

## Using It

If you're using wireguard-go you'll want to run it in a separate
terminal before running the `setup.sh` script. This command tells
wireguard-go to create a new wireguard interface called wg0.

```console
$ wireguard-go -f wg0
```

If you're using wireguard-go you'll want to run this export
command in the same terminal you're going to run `setup.sh`

```console
$ export WGGO=1
```

Here is the script you can name `setup.sh` to use to setup a
simple wireguard network.

```bash
#!/usr/bin/env bash
# Usage: sudo bash setup.sh wg.conf
set -xe

if [ "x$UID" != "x0" ]; then
  echo "You must be root to run this."
  exit 1
fi

# Check that the first argument is a file
if [ -f "${1}" ]; then
  # Set the conf variable to the first argument
  conf="${1}"
else
  # If it isn't a file then complain
  echo "Usage: sudo bash $0 wg.conf"
  exit 1
fi

# If you are using wireguard-go
# export WGGO=1
# before running this. You'll have to create the
# interface a different way described later
if [ "x${WGGO}" == "x" ]; then
  # Remove existing interface if it exists
  ip link del dev wg0 2>/dev/null || true
  # Create a new wireguard interface
  ip link add dev wg0 type wireguard
fi

# Parse out our IP from the config file
our_ip=$(cat "${conf}"| grep 'Address = ' | sed -e 's/.* = //g' -e 's/\/24//g')

# Tell Linux that our IP for the wireguard interface is the
# one we paresed out from the Address line in the config file
ip address add "$our_ip"/24 dev wg0

# Grab a list of our peers IPs by parsing every line
# with AllowedIPs in the config file
peers=$(cat "${conf}"| grep AllowedIPs | sed -e 's/.* = //g' -e 's/\/32//g')

for addr in ${peers}; do
  # Tell Linux that it can find our peers using the
  # interface we added
  ip address add dev wg0 "${our_ip}" peer "${addr}"
done

# Tell wireguard to use our config file
wg setconf wg0 "${conf}"

# Turn on the interface
ip link set up dev wg0

# Show the configured interface
ip addr show wg0
```

Make sure your config files are all correct, and then run
the script as root. The following example of running the
script includes sample output.

```console
$ sudo bash setup.sh wg.conf
+ '[' x0 '!=' x0 ']'
+ '[' -f wg.conf ']'
+ conf=wg.conf
+ '[' x == x ']'
+ ip link del dev wg0
+ ip link add dev wg0 type wireguard
++ cat wg.conf
++ sed -e 's/.* = //g' -e 's/\/24//g'
++ grep 'Address = '
+ our_ip=192.168.4.4
+ ip address add 192.168.4.4/24 dev wg0
++ cat wg.conf
++ sed -e 's/.* = //g' -e 's/\/32//g'
++ grep AllowedIPs
+ peers='192.168.4.115
192.168.4.3'
+ for addr in ${peers}
+ ip address add dev wg0 192.168.4.4 peer 192.168.4.115
+ for addr in ${peers}
+ ip address add dev wg0 192.168.4.4 peer 192.168.4.3
+ wg setconf wg0 wg.conf
+ ip link set up dev wg0
+ ip addr show wg0
25: wg0: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1420 qdisc noqueue state UNKNOWN group default qlen 1000
    link/none
    inet 192.168.4.4/24 scope global wg0
       valid_lft forever preferred_lft forever
    inet 192.168.4.4 peer 192.168.4.115/32 scope global wg0
       valid_lft forever preferred_lft forever
    inet 192.168.4.4 peer 192.168.4.3/32 scope global wg0
       valid_lft forever preferred_lft forever
```

On Windows, you'll just save the config file and click Activate.

Once everyone has run the script or activated the config (for
Windows). Then you can try pinging the other hosts.

> You may not be able to ping some Windows machines. But they
> will be able to ping you. So try the reverse when it's not
> working.

```console
$ ping 192.168.4.3
PING 192.168.4.3 (192.168.4.3): 56 data bytes
64 bytes from 192.168.4.3: icmp_seq=0 ttl=128 time=3.197 ms
64 bytes from 192.168.4.3: icmp_seq=1 ttl=128 time=3.617 ms
64 bytes from 192.168.4.3: icmp_seq=2 ttl=128 time=37.290 ms
64 bytes from 192.168.4.3: icmp_seq=3 ttl=128 time=3.504 ms
64 bytes from 192.168.4.3: icmp_seq=4 ttl=128 time=3.626 ms
64 bytes from 192.168.4.3: icmp_seq=5 ttl=128 time=37.651 ms
64 bytes from 192.168.4.3: icmp_seq=6 ttl=128 time=3.461 ms
^C--- 192.168.4.3 ping statistics ---
7 packets transmitted, 7 packets received, 0% packet loss
round-trip min/avg/max/stddev = 3.197/13.192/37.651/15.356 ms
```

## Gotchas

- If you see `ping: sending packet: Destination address required`.
  It's likely that everyone need to check their configs to make
  sure that everyone elses public keys are correct.

- The Windows client takes an `Address` parameter that's used as the
  IP address for the machine running that config. You'll need to
  uncomment it.

## References

- https://www.wireguard.com/quickstart/

- https://blog.jessfraz.com/post/installing-and-using-wireguard/

  - Example of using the demo server

- Scripts that run the wireguard demo server

  - https://git.zx2c4.com/wireguard-tools/tree/contrib/ncat-client-server

- https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing#IPv4_CIDR_blocks

  - The reason we do `/32` behind all the IPs of our peers is because
    it means there is only 1 address in that block, that one address
    is the address of our peer (the one preceding the `/`)

- https://en.wikipedia.org/wiki/Private_network#Private_IPv4_addresses

  - If you have more than 255 peers you want to connect then you'll
    want to change the `/24` to something appropriate. You'll have to
    read this and the previous link to figure out what you should use.
