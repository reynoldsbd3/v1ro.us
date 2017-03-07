---
author:
  description: Software Engineer
  email: reynoldsbd3@hotmail.com
  github: https://github.com/reynoldsbd3
  image: /img/avatar.png
  name: Bobby Reynolds
  website: https://www.reynoldsbd.net/
cardbackground: '#263238'
cardheaderimage: /img/wsl.jpg
cardthumbimage: /images/default.jpg
categories:
- post
date: 2017-02-27T15:09:00Z
description: Some tricks for making the most of the Windows Subsystem for Linux
tags:
- wsl
- windows
- zsh
title: WSL Hacks - Part 1
---

The Windows Subsystem for Linux, or WSL, is a new feature in Windows 10 that allows Linux binaries to be run unmodified
on a Windows system without using a VM or any other kind of emulation. Although technically still in Beta, WSL is
impressively stable and polished. For more information about WSL, including how to get started, see the
[official MSDN documentation](https://msdn.microsoft.com/en-us/commandline/wsl/about).

Out of the box, the WSL experience can feel a little minimal, especially compared to a heavily customized MacOS or Linux
system. And since WSL is a true Windows-native solution all the way down to the NT kernel, the process of customizing it
can seem daunting or downright obfuscated for somebody accustomed to Unix.

This is the first in a series of posts cataloging how I've made WSL into my own daily terminal driver. Some of these
tips may seem perfectly obvious to a seasoned Windows power user, and others might be child's play to the experienced
Linux sysadmin. Hopefully there's something here for everyone!


## ConHost and Fonts
---

At present, WSL is used through a console application called `bash.exe` which is hosted by Windows' built-in terminal
emulator `conhost.exe`. Although historically spartan in terms of customization, the amount of features added to ConHost
since the release of Windows 10 is impressive, and its future [looks bright](https://github.com/Microsoft/BashOnWindows/issues/111#issuecomment-238302654).

ConHost has just a few options for customization, and these can be accessed by right-clicking the title bar of a ConHost
window and choosing "Properties" from the context menu. The resulting dialog allows you to customize the appearance of
the console.

> **NOTE:** the settings in this dialog are associated with *the shortcut used to launch* the console application and
> *not* with the terminal emulator or the application itself. For example, if you have both a start menu entry and a
> taskbar shortcut for WSL, and you launch using the taskbar shortcut, any settings you change will *not* be reflected
> when you launch using the start menu, or when you launch another app like PowerShell.

![Console Properties Dialog](/img/console-properties-dialog.png)

By default, only a very few fonts are available, but it's easy to install more. For example, you could find a nice
monospace font from [Google Fonts](https://fonts.google.com/), download it, and install the `.ttf` file. Then, as long
as the font meets [certain criteria](https://support.microsoft.com/en-us/help/247815/necessary-criteria-for-fonts-to-be-available-in-a-command-window)
it will automagically be available in the ConHost properties dialog.

As you can (kinda) see in the image above, I've chosen to insall a [PowerLine-patched font](https://github.com/powerline/fonts)
to support some rich Zsh customization.


## Beyond Bash
---

Bash is nice, but I think Zsh is nicer. For [technical reasons](https://github.com/Microsoft/BashOnWindows/issues/846#issuecomment-238747413),
changing the default shell used by WSL is not as straightforward as using [`chsh`](https://linux.die.net/man/1/chsh).
There are several ways to work around this, but I've chosen to use the workaround presented [here](http://www.howtogeek.com/258518/how-to-use-zsh-or-another-shell-in-windows-10/).

In short, adding the following lines to `~/.bashrc` will *effectively* change your login shell to Zsh (assuming you've
already installed Zsh):

```bash
if [ -t 1 ]; then
        export SHELL=/usr/bin/bash
        exec /usr/bin/zsh
fi
```

This replaces the running `bash` shell with a new `zsh` shell. The `if [ -t 1 ]` makes sure this only happens if bash is
launched as an interactive shell, and the `export SHELL` prevents other applications from assuming that we prefer bash.

> Alternately, you could edit the Windows shortcut used to launch `bash.exe` and provide a `-c /usr/bin/zsh` argument,
> but I prefer to piggyback off of `.bashrc` because it gives me an extra place to put WSL-specific startup commands.

At this point, you're free to customize Zsh to you're liking. I'm using [Oh My Zsh](https://github.com/robbyrussell/oh-my-zsh)
with the [Bullet Train](https://github.com/caiogondim/bullet-train.zsh) theme:

![Console Screenshot](/img/console-screenshot.png)

We can't yet customize the colors used by ConHost (e.g. to make a [Solarized](http://ethanschoonover.com/solarized)
theme), but full 24-bit colors is [in the pipeline](https://blogs.msdn.microsoft.com/commandline/2016/09/22/24-bit-color-in-the-windows-console/).
Also, a few of the PowerLine font characters are just plain not rendered, but not enough to really bother me.


## Filling the Gaps
---

Again, WSL is *not* a full-fledged Linux system; it's just a shim on top of the NT kernel. Many of the creature comforts
you'd expect to have in another Unixy environment just aren't there, but there are steps we can take to make WSL feel
a little more like home.

Because of the way I'm changing my shell to Zsh (by swapping out at the end of my `.bashrc`), I can use the `.bashrc` to
take care of initialization that only needs to happen on WSL, leaving my `.zshrc` happily platform agnostic (I happen to
use the exact same `.zshrc` on *real* Linux and BSD systems).


### X Windows

Although none of the official documentation makes any mention of it, WSL can indeed be used to run graphical Linux
applications pretty much seamlessly. This is thanks to the fundamentally networked nature of X11. The only thing you
need to do is install an X server for Windows. I reccomend [Xming](http://www.straightrunning.com/XmingNotes/).

Then set `$DISPLAY` and everything should Just Work:

```bash
export DISPLAY=localhost:0.0
```


### Cleaning out `/tmp`

The `/tmp` directory on a Unix system is a place for programs to put temporary files. There is no guarantee that files
will still be there after a reboot, and most Linux distro's actually mount `/tmp` as a RAM-backed `tmpfs` device for
performance.

WSL does not do this; instead, `/tmp` is just part of the root file system. There is nothing technically wrong with
this, but my personal preference is to have `/tmp` purged frequently so that temporary files don't take up too much
space.

I could just put this in my `.bashrc`:

```bash
rm -rf /tmp/*
```

But that's risky. What if there's another Linux process running (in another WSL terminal) that's using something in
there? Ideally, I would want `/tmp` to act just like a `tmpfs` and lose everything only when the OS shuts down.

To accomplish this, I can use `/dev/shm`, which IS a `tmpfs` file system. In WSL, `/dev/shm` is maintained as long as
there is at least one `bash.exe` running, and then it's wiped. So, I'll add the following to my `.bashrc`:

```bash
if [ ! -f "/dev/shm/wsl-session" ]; then
        
        touch /dev/shm/wsl-session

        rm -rf /tmp/*
fi
```

This will delete everything in `/tmp` the first time I start Bash, but leave things untouched if I'm just opening a
second terminal. Incidentally, I can put whatever I want in this `if` block, and it will be run on a session-by-session
basis instead of terminal-by-terminal.


### SSH Agent

Another convenience missing from WSL is a persistent SSH agent. Using the same technique as above, I can start up
`ssh-agent` once per session and have each terminal automatically use it:

```bash
if [ ! -f "/dev/shm/wsl-session" ]; then
        
        # ... other session initialization

        # Fire up ssh-agent once per session
        mkdir /dev/shm/ssh-agent
        ssh-agent -a /dev/shm/ssh-agent/agent.sock &> /dev/shm/ssh-agent/env
fi

# Every time a terminal starts, source the proper environment variables
. /dev/shm/ssh-agent/env > /dev/null
```
