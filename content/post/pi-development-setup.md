---
author:
  description: Software Engineer
  email: reynoldsbd3@hotmail.com
  github: https://github.com/reynoldsbd3
  image: /img/avatar.png
  name: Bobby Reynolds
  twitter: https://twitter.com/
  website: http://v1ro.us/
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

I am using a Raspberry Pi 2, model B, revision 1.1 for my development work.
However, these tips and tricks should apply to all Pi's. For a serial
connection, I'm using this USB serial cable from Adafruit.

Also, I do all of my development on a Linux virtual machine running inside
VMWare Workstation on a Windows host. Believe it or not, this works great!
Things like SD card formatting and even serial communication work with zero
configuration because VMWare and Windows just pass the USB devices through to
Linux.

## Serial Console

When doing low-level stuff, it's pretty essential to have a serial connection
for debugging the code and getting output. There are quite a few programs out
there that can be used for serial communication: minicom, screen, dterm...

I recently stumbled upon a lesser known program and protocol called Kermit that
actually fills my needs quite nicely. It is flexible and provides some useful
features like file transfer (see the next section for more information about
that).

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
