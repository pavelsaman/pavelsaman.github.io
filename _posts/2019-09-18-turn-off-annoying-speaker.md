---
layout: post
title:  "Turn Off Annoying Speaker"
date:   2019-09-18 20:07:19 +0200
categories: linux
---
Have you ever experienced this annoying beep sound your PC speaker makes on error, in console and possibly on other places? This is the single most annoying thing I wanted to get rid off system-wide and for good. If you want to do the same, this short blog post might come in handy.

There are a few different places where this could be disabled, but I wanted to do it system-wide, so I don't need to worry about all the places where to turn it off.

It turns out that the speaker functions because of `pcspkr` kernel module. Knowing this, I only created a new file in `/etc/modprobe.d/` with the following content:

```bash
$ cat blacklist-mine.conf 
blacklist pcspkr
```

This will not load the kernel module on next boot, thus silencing the PC speaker :)

Of course, the same method could be used if you don't want to load some other kernel modules. One way to find out what piece of hardware uses what kernel module is command `lspci -v`, the output looks like this:

```bash
$ lspci -v
00:00.0 Host bridge: Intel Corporation Broadwell-U Host Bridge -OPI (rev 09)
	Subsystem: Dell Broadwell-U Host Bridge -OPI
	Flags: bus master, fast devsel, latency 0
	Capabilities: <access denied>
	Kernel driver in use: bdw_uncore
......
......
```

The last line with no dots shows what kernel module is used.