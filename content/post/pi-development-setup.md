---
author:
  description: Software Engineer
  email: reynoldsbd3@hotmail.com
  github: https://github.com/reynoldsbd3
  image: /img/avatar.png
  name: Bobby Reynolds
  twitter: https://twitter.com/
  website: http://reynoldsbd.net/
cardbackground: '#263238'
cardheaderimage: /images/default.jpg
cardthumbimage: /images/default.jpg
categories:
- post
date: 2016-03-01T22:11:01-05:00
description: Post Description
tags:
- meta
- test
title: Raspberry Pi Bare-Metal Workflow
---

I like to do lots of low-level stuff with my Raspberry Pi, such as kernel
development and other bare-metal programming. Briefly, this means writing
assembly and C code to be run on the Pi outside of Linux on the CPU itself.
Doing this is fun and challenging, but there is a lot of overhead work required
to compile code, transfer it to the Pi, and get it to boot.

I've come up with a few tricks that streamline different parts of the
development process. I find that these methods make the process of developing
low-level stuff much easier.

## Background

I am using a [Raspberry Pi 2, model B][TODO] for my development work. However,
these tips and tricks should apply to all Pi's.

All of my development is done in Elementary OS, which is an Ubuntu Linux
derivative. This system is actually a virtual machine running inside
[VMWare Workstation][TODO] on a Windows host. Believe it or not, this works
great! Things like SD card formatting and even serial communication work with
zero configuration because VMWare and Windows just pass the USB devices through
to Linux.

However, Linux is Linux, so feel free to use whatever distribution suits you.

## Serial Console

[Serial communication][TODO] is a must when doing low-level and bare metal development.
Without it, getting output from a program requires a working driver for some
other output device, and it's pretty hard to write a driver for an output device
without being able to see any output!

### Hardware

I use [this USB-to-serial][TODO] adapter from Adafruit to connect to my Pi.
Honestly, though, I've had a pretty bad experience with this particular model.
It's poorly manufactured and easily breaks, and about half it severely corrupts
the data it's transmitting. More recently, sending data has simply stopped
working! I'm not sure if this is due to a short in the TX line or simply a bad
controller getting worse.

But I digress; this cable sucks and you should shop around for an alternative.

### Software

There are several popular programs used to communicate over a serial connection.
[Gnu Screen][TODO] and [Minicom][TODO] are two popular options, but there are
many more out there. I recently stumbled upon a lesser known program and
protocol called [Kermit][TODO] that fills my needs quite nicely. It is flexible
and provides some extra features that I'll discuss later.

Kermit is probably available in most Linux distribution repositories by default,
though probably under the name `ckermit`. So, installing it is pretty simple:

```bash
# Debian/Ubuntu
sudo apt-get install ckermit

# Fedora/CentOS/RHEL
sudo yum install ckermit

# Arch
sudo pacman -S ckermit
```

If you run just plain `kermit`, you'll see that it's actually kind of like an
interpreter. This is where you access most of kermit's functionality beyond
simply using it as a serial console. For example, typing `send /path/to/file`
from this prompt will send a file to the connected device using the kermit
protocol.

Rather than researching the exact commands and options needed to connect to a
Raspberry Pi and entering them each time, configuration can be written to the
file `~/.kermrc`, which is read and executed each time kermit is started.

Here's a copy of my `.kermrc` file:

```bash
set line /dev/ttyUSB0
set speed 115200
set carrier-watch off
set prefixing all
set parity none
set stop-bits 1
set modem none
set file type bin
set file name lit
set flow-control none
set prompt "kermit> "
```

This configuration sets kermit up so that each time it's started it connects to
the serial connection at `/dev/ttyUSB0` with a baud rate of 115200. Several
other options for the connection are set, most of which I honestly don't
understand, so I won't try to explain them!

In order to access serial devices in Linux *without* being root or using sudo,
your user account must be added to the appropriate group, and you must log out
and back in for the group change to take effect:

```bash
# Debian/Ubuntu
sudo usermod -aG dialout $(whoami)

# Other
sudo usermod -aG uucp $(whoami)
```

The kermit command `connect` (or `c` for short) is used to establish a serial
console with the Raspberry Pi. It only works if kermit has been told which line
to use for the serial connection (see the kermrc above). Check it out:

```bash
user@home ~ $ ckermit
C-Kermit 9.0.302 OPEN SOURCE:, 20 Aug 2011, for Linux (64-bit)
 Copyright (C) 1985, 2011,
  Trustees of Columbia University in the City of New York.
Type ? or HELP for help.
kermit> connect

user@rpi ~ $ ^C
user@rpi ~ $ ^C
user@rpi ~ $ ^C
user@rpi ~ $

kermit> quit
```

The key combination `Ctrl+\` followed by `C` will drop back to the kermit
command line, and the kermit command `quit` will exit kermit.
