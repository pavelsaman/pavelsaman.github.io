---
layout: page
title:  "Linux Kernel Compilation and $KERNELRELEASE"
date:   2019-12-14 010:07:19 +0200
categories: linux
---

Out of curiosity, I've told myself I'd try to compile my own custom kernel. Not that I have any particular need for it, but I just wanted to know how it's done without using modern methods such as running package managers to do the job for you. After quite some time, I eventually succeeded, but what I want to share in this article is how to investigate some errors that might occur and how to name your custom kernel version.

### Where To Find Information

First of all, the Internet is full of tutorials on how to do it, you can e.g. follow [this article on Arch wiki](https://wiki.archlinux.org/index.php/Kernel/Traditional_compilation). I'd recommend following this one over many other ones that don't go into such a detail.

### Getting Kernel Config

One of the first decisions you're gonna face when compiling your own kernel is where you get a kernel config. There're a few options:

- you can get a default one from the Internet
- you can use the one currently in use with your running kernel (if enabled) ![image](/images/config_enabled.png)
- you can set up every bit on your own issuing `make menuconfig`
- running `make localmodconfig` that will generate a configuration of your running kernel; but it will disable all options not currently in use, so you might find out pretty soon that some devices not plugged in when running this command simply don't work because they don't have the necessary support in your new kernel
- and possibly some other options

In my case, I went for the second option with command `zcat /proc/config.gz > .config`, and then I played around with some other settings with `make menuconfig` (mostly to see what's possible and to see how much I don't know about all the possible settings).

### Errors

Ok, now the interesting part. After the kernel compilation, I created a new `/etc/mkinitcpio.d/linux543.preset` and executed `mkinitcpio -p linux543` command. But there was the following error:

```bash
$ sudo mkinitcpio -p linux543
==> Building image from preset: /etc/mkinitcpio.d/linux543.preset: 'default'
  -> -k /boot/vmlinuz-linux543 -c /etc/mkinitcpio.conf -g /boot/initramfs-linux543.img
==> ERROR: invalid kernel specified: `/boot/vmlinuz-linux543'
==> Building image from preset: /etc/mkinitcpio.d/linux543.preset: 'fallback'
  -> -k /boot/vmlinuz-linux543 -c /etc/mkinitcpio.conf -g /boot/initramfs-linux543-fallback.img -S autodetect
==> ERROR: invalid kernel specified: `/boot/vmlinuz-linux543'
```

**Lesson #1:** you really need to copy the correct file after compilation :D And yes, it took me a few minutes to find out I had a wrong file. So after running:

```bash
$ sudo cp -v bzImage /boot/vmlinuz-linux543 
'bzImage' -> '/boot/vmlinuz-linux543'
```

I finally made this error go away.

However, my next surprise was this error:

```
==> Starting build: linux-5.4.3
  -> Running build hook: [base]
  -> Running build hook: [udev]
  -> Running build hook: [autodetect]
  -> Running build hook: [modconf]
  -> Running build hook: [block]
  -> Running build hook: [keyboard]
==> ERROR: module not found: `usbhid'
  -> Running build hook: [keymap]
  -> Running build hook: [resume]
  -> Running build hook: [filesystems]
==> WARNING: No modules were added to the image. This is probably not what you want.
==> Creating gzip-compressed initcpio image: /boot/initramfs-linux543.img
==> WARNING: errors were encountered during the build. The image may not be complete.
```

Ok, this looked a bit easier to figure out, seemed like I'd forgotten to prepare my modules. So going a step back, I executed:

```bash
$ sudo make modules_install 
ln: target 'KERNEL/source': No such file or directory
make: *** [Makefile:1315: _modinst_] Error 1
```

It was not good, I had no idea what to do with this error for quite a while. First, I googled, but didn't really find much, so the only option was to find out more on my laptop, which would've allowed me to make my search queries more precise.

### $KERNELRELEASE

The obvious next step was to have a look inside `Makefile`, the error occured on line 1315 after all. In my case, it looked like this:

```
PHONY += _modinst_
_modinst_:
	@rm -rf $(MODLIB)/kernel
	@rm -f $(MODLIB)/source
	@mkdir -p $(MODLIB)/kernel
	@ln -s $(abspath $(srctree)) $(MODLIB)/source
	@if [ ! $(objtree) -ef  $(MODLIB)/build ]; then \
		rm -f $(MODLIB)/build ; \
		ln -s $(CURDIR) $(MODLIB)/build ; \
```

And like 1315 was exactly this one:

```
@ln -s $(abspath $(srctree)) $(MODLIB)/source
```

Ok, this tries to create a symlink `$(MODLIB)/source`. By this point, I'd already narrowed down my searches and been able to find out some forum posts that mentioned similar errors. Based on that I investigated what exactly is `MODLIB` variable.

Makefile defines it like this:

```
MODLIB	= $(INSTALL_MOD_PATH)/lib/modules/$(KERNELRELEASE)
```

In my case `INSTALL_MOD_PATH` was defaulted to `/`, so I focused on `KERNELRELEASE`, which is defined like this:

```
KERNELRELEASE = $(shell cat include/config/kernel.release 2> /dev/null)
```

Upon seeing the result of `cat include/config/kernel.release` it was obvious where I made a mistake. It turned out I tweaked this config option:

![image](/images/kernelversion_wrong.png)

This wrong choice of a name saved spaces into `KERNELVERSION`, so the `ln` command saw that as more parameters, not as one path parameter.

So changing it back to something like:

![image](/images/kernelversion_ok.png)

meant my `KERNELVERSION` was equal to `5.4.3MANJARO_CUSTOM` and I managed to build the initramfs:

```bash
sudo mkinitcpio -p linux543
==> Building image from preset: /etc/mkinitcpio.d/linux543.preset: 'default'
  -> -k /boot/vmlinuz-linux543 -c /etc/mkinitcpio.conf -g /boot/initramfs-linux543.img
==> Starting build: 5.4.3MANJARO_CUSTOM
  -> Running build hook: [base]
  -> Running build hook: [udev]
  -> Running build hook: [autodetect]
  -> Running build hook: [modconf]
  -> Running build hook: [block]
  -> Running build hook: [keyboard]
  -> Running build hook: [keymap]
  -> Running build hook: [resume]
  -> Running build hook: [filesystems]
==> Generating module dependencies
==> Creating gzip-compressed initcpio image: /boot/initramfs-linux543.img
==> Image generation successful
==> Building image from preset: /etc/mkinitcpio.d/linux543.preset: 'fallback'
  -> -k /boot/vmlinuz-linux543 -c /etc/mkinitcpio.conf -g /boot/initramfs-linux543-fallback.img -S autodetect
==> Starting build: 5.4.3MANJARO_CUSTOM
  -> Running build hook: [base]
  -> Running build hook: [udev]
  -> Running build hook: [modconf]
  -> Running build hook: [block]
  -> Running build hook: [keyboard]
  -> Running build hook: [keymap]
  -> Running build hook: [resume]
  -> Running build hook: [filesystems]
==> Generating module dependencies
==> Creating gzip-compressed initcpio image: /boot/initramfs-linux543-fallback.img
```

### Conclusion

It's really good to have so many articles with 8 or so easy steps to compile your own kernel. But the reality usually is that something goes wrong. If the article is short, you might be left on your own. Sooner or later, you'll need to do the hard work yourself and dig into the problem. That's usually the only path to solution. In Linux, the good thing is, you can look things up easily, as in here when looking into Makefile which was the key step to resolving the problem.