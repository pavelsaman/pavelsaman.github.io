---
layout: page
title:  "Upgrading BIOS on Dell Laptop"
date:   2019-09-25 15:07:19 +0200
categories: linux
---

Just a quick article on how to find out BIOS version and how to get an update, and finally how to upgrade it.

I've recently done exactly this on my Dell E5550 laptop.

First, my BIOS version was A06:

```bash
$ sudo dmidecode -s bios-version
A06
```

Next question is how to find out if there's a new version for my laptop. I had to first get a serial number of my machine:

```bash
$ sudo dmidecode -s system-serial-number
xxxxx # don't want to make it public
```

Then on [Dell website](https://www.dell.com/support/home/uk/en/ukdhs1), I copy-pasted my serial number and let the system find new updates.

Surely, there was a new BIOS version `A21`.

It's always an `.exe` file, no matter what OS you have. There's also a good tutorial on how to upgrade BIOS on Linux machines. The tutorial there is for Ubuntu since that's an OS Dell delivers on its machines.

Then, I had to do a few more things:

1. I downloaded the `.exe` file
2. I verified checksums of the downloaded file against what Dell has on their website
3. I formatted a usb flash drive, it's important to use FAT 32
In my case, I used `fsdisk` and `mkfs.fat` tools, but it's possible to use some other tools, e.g. GUI tools present in your distro.
4. I copied the `.exe` file onto my flash drive

Then it's all about rebooting, pressing `F12` and selecting `BIOS Flash Update`. The rest should be automatic and if there're no problems, the BIOS should be upgraded to a new version (in my case A21), which I can confirm either in BIOS setup or after boot in my OS:

```bash
$ sudo dmidecode -s bios-version
A21
```