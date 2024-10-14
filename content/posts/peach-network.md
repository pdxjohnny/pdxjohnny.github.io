+++
title = "Fuzzing network applications with peach"
date = "2016-08-10T13:35:13-07:00"

+++

So you want to fuzz network applications with peach eh? Well you've come to the
right place. This is a tutorial on how to get you fuzzing TCP applications,
without TLS/SSL enabled. If you want to fuzz UDP or an application which only
communicates via TLS/SSL then this is a great place for you to start, however it
will not answer all your questions. I will point you in the right direction at
the end of this post.

The first step in fuzzing is to understand the structure of the protocol. It
often helps to have an example of this structure. Therefore we will be
capturing the data of our target and simply playing it back. If you were to
spend more time, which you should, then you would make data models in peach
which contain specific fields rather than the blob we will be using.

The two most well known ways of getting network traffic are tcpdump and
wireshark. Peach can use input from a file for the data model and mutate it. We
are going to use a tiny tool I wrote to capture the conversations back and
forth rather than telling you to open tcpdump / wireshark and copy paste to a
file. If you would rather do that be my guest.

## Enter convo-capture

This saves the TCP conversation to files. Give it the port and host it needs to
be monitoring. This is especially useful for fuzzing with peach on TCP based
programs so that you don't have to go into wireshark to capture then copy paste
the data from each sequence of packets. This will take all packets and put them
in a file until the other endpoint sends data. Then it will increment the
number on the exchange and write the data to that exchanges file.

The binary has been built and included in the gist for your convenience.

> By no means do I condone putting binaries in git repos but I know not
> everyone has the go toolchain. You need libpcap and libpthread to run it.

Building will use sudo because it will `set_cap_raw` on convo-capture. If you
don't want to set this then you have to run convo-capture as root.

```bash
git clone https://gist.github.com/pdxjohnny/e2d1df77e81f07254da192fe1bc568a0 convo-capture
cd !$
./build.sh
```

Now you can move it to bin if you want or run it prepended with ./ from the
directory you built it.

```bash
sudo mv convo-capture /usr/bin/convo-capture
```
> Personally I like to put things in ~/.local/bin/ but do as you will

We are now ready to capture packets. Keep in mind that convo-capture will not
write over files that you have previously captured if they are in the directory
you are working in. Be sure to delete files from previous captures or change to
a new directory.

Let's try to capture HTTP traffic using curl and python3's http.server
(SimpleHTTPServer in python2). First we need to ssh to another computer and
start an HTTP server or we can start one on our local machine. If you start one
on your machine then everywhere you see example.com replace it with localhost
and add `-i lo` to convo-capture for capturing on the loopback interface.

```bash
[pdxjohnny@pdxjohnny convo-capture]$ ssh example.com
[pdxjohnny@example.com ~]$ python3 -m http.server 4444
```

Now that we have a HTTP server running lets start convo-capture.

```bash
# For localhost add -i lo
# convo-capture -p 4444 -ip localhost -i lo -v
[pdxjohnny@pdxjohnny convo-capture]$ convo-capture -p 4444 -ip example.com -v
Capturing TCP port 4444 for host example.com
```

Ok great we are now capturing all traffic going to example.com on port 4444 from
our computer and from example.com port 4444 back to our computer.
Now we just need to use curl to generate a request we can capture.

```bash
curl -v http://example.com:4444/file
```

Request sent! Look back at the session running convo-capture, you should see
that the output you observed in curl has been captured (expect for the < and >
left of the headers, curl adds those).

```log
[pdxjohnny@pdxjohnny convo-capture]$ convo-capture -p 4444 -ip example.com -v
Capturing TCP port 4444 for host example.com
GET /file HTTP/1.1
Host: example.com:4444
User-Agent: curl/7.47.0
Accept: */*

HTTP/1.0 200 OK
Server: SimpleHTTP/0.6 Python/3.4.2
Date: Wed, 10 Aug 2016 16:38:42 GMT
Content-type: text/plain
Content-Length: 514
Last-Modified: Tue, 26 Jul 2016 14:43:05 GMT

Yo this is the file
```

Now you can ctrl-c to stop the capture. As you can see we captured the
conversation from our local machine to the remote host and the response the
remote host sent us. If you do an ls you will also see the files that were
created by this capture.

```log
[pdxjohnny@pdxjohnny convo-capture]$ ls -lAF
total 5752
... Aug 10 09:34 10.7.202.149->10.7.202.78-0
... Aug 10 09:34 10.7.202.78->10.7.202.149-0
... Aug 10 08:35 build.sh*
... Aug 10 09:17 convo-capture*
... Aug 10 09:33 .git/
... Aug 10 09:16 .gitignore
... Aug 10 09:15 main.go
... Aug 10 09:58 README.md
```

cating the files will make convo-capture's usefulness apparent.

```log
[pdxjohnny@pdxjohnny convo-capture]$ cat 10.7.202.149->10.7.202.78-0
GET /file HTTP/1.1
Host: example.com:4444
User-Agent: curl/7.47.0
Accept: */*

[pdxjohnny@pdxjohnny convo-capture]$ cat 10.7.202.78->10.7.202.149-0
HTTP/1.0 200 OK
Server: SimpleHTTP/0.6 Python/3.4.2
Date: Wed, 10 Aug 2016 16:38:42 GMT
Content-type: text/plain
Content-Length: 514
Last-Modified: Tue, 26 Jul 2016 14:43:05 GMT

Yo this is the file
```

As you can see it has assembled the packets into files based on the order they
were sent in. For me the second file, the servers reply, was two packets.
convo-capture saw the two packets in a row from the server to client and said
ok this is all part of one message I'm going to save it to a file as such. A
message is a continuous sequence of packets ended when the other side starts
sending a message. The more messages that are collected the more files you will
see after you kill convo-capture.

There was only one back and forth so they are both 0 in the sequence. If you
were to have ran curl twice with convo-capture running then you would see the
contents of 0 repeated in 1.

This is very useful for fuzzing with peach. Peach allows us to order our call
and response to the target program. For example say you want to fuzz something
like git. git is not a simple call and response. It has an exchange of call,
response, call, response for a clone. Let's walk through how you would use
convo-capture to fuzz the git protocol with peach.

## Capturing the git protocol

[![asciicast](https://asciinema.org/a/7xd0u4u0vfv7n7s7gq4f0zdg2.png)](https://asciinema.org/a/7xd0u4u0vfv7n7s7gq4f0zdg2)

> This video goes shows the how to capture the data involved in a git clone
> over the git protocol. Use it as a reference if you are having trouble with
> the steps below.

Let's take a stroll on over to tmp so we don't create a bunch of useless files.
We'll make a directory there to play in.

```bash
mkdir /tmp/demo
cd /tmp/demo
```

Make a few directories so nothing writes over each other. Then well make a git
repo and populate it with some files.

```bash
mkdir clonedir capture
mkdir testrepo
cd testrepo
cat << EOF > README.md
This is a super cool test repo
EOF
git init
git add -A
git commit -sam 'Added README.md'
cd ..
# You should now be back in /tmp/demo
```

We have our testrepo, now lets create a bare copy of it to be served by [git
daemon][git-daemon]. This requires that we make the `git-daemon-export-ok` file
as well. Then we will start the git server.

```bash
git clone --bare testrepo testrepo.git
touch testrepo.git/git-daemon-export-ok
git daemon --reuseaddr --base-path=$PWD $PWD
# PWD is faster than typing /tmp/demo
```

Great! The git server is up! We can look up what port it is running on, but
if perhaps we were fuzzing something we didn't know we would have to find out.

```bash
# Apparently netstat is deprecated, so let's use ss
ss -ltnp | grep git
```

Alright its port 9418. In another shell go to the capture directory and start
convo-capture.

```bash
cd /tmp/demo/capture
convo-capture -p 9418 -i lo
```

Capture is running, git server is up, all that's left is to go to clonedir and
watch the magic happen.

```bash
cd /tmp/demo/clonedir
git clone git://localhost/testrepo.git
```

Now switch back to the shell running convo-capture and hit it will ctrl-c. You
should see four files in the capture directory. We are going to fuzz the git
client so right now we are interested in the files which go from port 9418 to
some other port.

## Using our captured data to fuzz with peach

You should usually test against the master branch or the latest version of
whatever you are fuzzing. You don't want to waste time finding something which
has already been fixed. This is why we are going to build git from source. Of
course you don't have to do this. But if you have never built something from
source it would be good practice.

[![asciicast](https://asciinema.org/a/82390.png)](https://asciinema.org/a/82390)

> This video shows the peach process. You probably want to skip past the
> part were we run make on git.

Now that we know what the git server is sending to the client we could either
fuzz the server or the client. The client is easy because it exits after
cloning were as the server stays up to serve requests.

We are going to compile git from source so we need to download its
dependencies.

```bash
sudo apt-get -y install libcurl4-gnutls-dev libexpat1-dev gettext \
  libz-dev libssl-dev \
  || sudo yum install curl-devel expat-devel gettext-devel \
  openssl-devel perl-devel zlib-devel
```

Now we are going to clone git build it and install it.

```bash
mkdir -p /tmp/demo/
cd /tmp/demo/
git clone --depth=1 https://github.com/git/git
cd /tmp/demo/git/
# I found that the latest git doesn't cooperate unless I install it
sudo make install
git --version
```

Git is installed, now lets copy our relevant captures to a testing directory.
Here we clone the repo for this post and copy the git.xml file out of it. But
you could of course make your own or modify the one here.

```bash
cd /tmp/demo/
mkdir gitfuzzy
cd gitfuzzy
cp /tmp/demo/capture/\:\:1\:9418-\>* ./
git clone https://gist.github.com/pdxjohnny/e2d1df77e81f07254da192fe1bc568a0 t
cp t/git.xml ./
```

`git.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<Peach>
    <DataModel name="TheDataModel">
        <Blob/>
    </DataModel>
    <StateModel name="TheState" initialState="Initial">
        <State name="Initial">
            <Action type="accept"/>
            <!-- receive bytes -->
            <Action type="input">
                <DataModel ref="TheDataModel"/>
            </Action>
            <!-- send bytes -->
            <Action type="output">
                <DataModel ref="TheDataModel"/>
                <!-- Change this to be whatever port your git client was on -->
                <Data fileName="::1:9418->::1:58226-0"/>
            </Action>
            <!-- receive bytes -->
            <Action type="input">
                <DataModel ref="TheDataModel"/>
            </Action>
            <!-- send bytes -->
            <Action type="output">
                <DataModel ref="TheDataModel"/>
                <!-- Change this to be whatever port your git client was on -->
                <Data fileName="::1:9418->::1:58226-1"/>
            </Action>
        </State>
    </StateModel>
    <Agent name="LinAgent">
        <!-- Register for core file notifications. -->
        <Monitor class="LinuxDebugger">
            <!-- This is the program we're going to run inside of the debugger -->
            <Param name="Executable" value="git"/>
            <!-- These are arguments to the executable we want to run -->
            <Param name="Arguments" value="clone git://127.0.0.1/testrepo.git"/>
            <!-- This parameter will cause the monitor to terminate the process
								 once the CPU usage reaches zero. -->
            <Param name="CpuKill" value="true"/>
        </Monitor>
        <Monitor class="CleanupFolder">
            <Param name="Folder" value="testrepo"/>
        </Monitor>
    </Agent>
    <Test name="Default">
        <Agent ref="LinAgent" platform="linux"/>
        <StateModel ref="TheState"/>
        <Publisher class="TcpListener">
            <Param name="Interface" value="127.0.0.1"/>
            <Param name="Port" value="9418"/>
        </Publisher>
        <Strategy class="Random"/>
        <Logger class="Filesystem">
            <Param name="Path" value="logs"/>
        </Logger>
    </Test>
</Peach>
```

Your git client used a different port to connect to the git server when we did
the capture than mine did. When you copy the xml file you will have to change
the values as indicated with comments so that peach knows what files to use.

```bash
# Change to the correct files
vim git.xml
peach git.xml
```

Peach is fuzzing the git protocol now! Good job you rock!

[git-daemon]: https://git-scm.com/book/en/v1/Git-on-the-Server-Git-Daemon
