---
layout: page
title:  "Bootable USB Flash Drive With Linux Tools"
date:   2019-12-14 17:07:19 +0200
categories: linux
---

It's really popular to use external tools such as [balenaEtcher](https://www.balena.io/etcher/) to flash OS images. Although it's a good tool, you can accomplish the same task with tools that are already present in Linux systems. Let's have a look what I mean.

### Prepare Flash Drive

First thing you need to get a flash driver, plug it in and **unmount** it. Yes, many OS will mount your flash drive automatically via different tools, but for the following procedure, it needs to be unmounted.

You might end up with a similar situation.

```bash
$ lsblk
NAME                                          MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
sda                                             8:0    0   477G  0 disk  
├─sda1                                          8:1    0   512M  0 part  /boot/efi
├─sda2                                          8:2    0     8G  0 part  [SWAP]
├─sda3                                          8:3    0  87.9G  0 part  /
└─sda4                                          8:4    0 371.1G  0 part  
  └─luks-11e54aef-4128-4301-9e9a-230c7dbfe1a9 254:0    0 371.1G  0 crypt /home
sdb                                             8:16   1   3.8G  0 disk  
├─sdb1                                          8:17   1   2.7G  0 part  
└─sdb2                                          8:18   1     4M  0 part
```

In my example, `sdb` is the name of my flash drive. Make sure you get the correct name since the following operation will destroy data on your device.

### Copy OS Image

Now after you've donwloaded and verifier (gpg, hashes) an iso file of your desired OS (linux), you need to copy it onto the flash drive. It turns out the only two tools you need are `dd` and `sync`:

```bash
$ sudo dd if=/path/to/my/iso of=/dev/sdb bs=4M status=progress && sync
```

What's going on here is this:

- `dd` will copy the data in 4M chunks (`bs=4M`) into `/dev/sdb`; **it's crucial you don't make a mistake, sdb and sda are close enough, but the latter would have a disastrous effect (in my case)**; `status=progress` is just for displaying some information about the whole write operation
- `sync` is important so it really waits until the write operation finishes, it makes sure all cached data are flushed onto the device

And that's it, your flash drive is ready to be booted from. No need for any external tool you'd need to install first.