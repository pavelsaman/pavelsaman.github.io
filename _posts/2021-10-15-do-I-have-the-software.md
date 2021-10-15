---
layout: page
title: "Do I Have the Software?"
date: 2021-10-15 22:15 +0200
categories: linux
---

One of the first things one wants to find out on a system is what is already installed. Do I have the software is a question we often want answers for. How to find out on Linux?

In short, there are a couple of ways:

1. `which` command
1. `whereis` command
1. `locate` and `updatedb`
1. package manager

These do not necessarily yield the same result, so you might need to combine them in order to find the information you need.

Let's have a quick look on each:

## `which`

`which` command searches for executables in `PATH`. That is enough when looking for binaries that are typically in `PATH` directories.

## `whereis`

This command searches more places than just `PATH`, the man page mentions:

> By default whereis tries to find files from hard-coded paths, which are defined with glob patterns. The command attempts to use the contents of $PATH and $MANPATH environment variables as default search path.

Therefore it could be a good extension of `which`.

## `locate` and `updatedb`

These two are, at least on my Artix system, part of `mlocate` package. The way it works is `updatedb` creates a database, and `locate` searches it. `updatedb` has its config file in `/etc/updatedb.conf` that can specify what places, names, etc. should not be taken into account when building the database. The file could look like this:

```
PRUNE_BIND_MOUNTS = "yes"
PRUNEFS = "9p afs anon_inodefs auto autofs bdev binfmt_misc cgroup cifs coda configfs cpuset cramfs debugfs devpts devtmpfs ecryptfs exofs ftpfs fuse fuse.encfs fuse.s3fs fuse.sshfs fusectl gfs gfs2 hugetlbfs inotifyfs iso9660 jffs2 lustre mqueue ncpfs nfs nfs4 nfsd pipefs proc ramfs rootfs rpc_pipefs securityfs selinuxfs sfs shfs smbfs sockfs sshfs sysfs tmpfs ubifs udf usbfs vboxsf"
PRUNENAMES = ".git .hg .svn"
PRUNEPATHS = "/afs /media /mnt /net /sfs /tmp /udev /var/cache /var/lib/pacman/local /var/lock /var/run /var/spool /var/tmp"
```

An important note here: if `updatedb` does not run, `locate` might not find a file even when it is present on the system. Therefore it is important to run `updatedb` periodically, e.g. with cron.

I also run `updatedb` after every update done by `pacman`:

```
$ cat /etc/pacman.d/hooks/10-updatedb.hook 
[Trigger]
Operation=Install
Operation=Remove
Type=Package
Target=*

[Action]
When=PostTransaction
Exec=/usr/bin/updatedb

```

In combination with daily cron, it means the database is more or less up to date most of the time, so `locate` command can do its job.

## package manager

Lastly, package managers usually have a way of finding out what packages are currently installed on a system.

An example with pacman could be:

```
$ pacman -Qs mlocate
local/mlocate 0.26.git.20170220-7
    Merging locate/updatedb implementation
```

`-Qs` parameters mean I can search for local packages. `-s` parameter takes a regex, `-Q` stands for query, that is to query the local database.

```
$ pacman -Q
```

This command will return all installed packages, it's quite a list:

```
$ pacman -Q | wc -l
849
```

Pacman can also work with package files, e.g. to find out about package files, I can execute:

```
$ pacman -Qk telegram-desktop
telegram-desktop: 31 total files, 0 missing files
```

and to see more specifically all files owned by a package:

```
$ pacman -Ql telegram-desktop
telegram-desktop /usr/
telegram-desktop /usr/bin/
telegram-desktop /usr/bin/telegram-desktop
telegram-desktop /usr/share/
telegram-desktop /usr/share/applications/
telegram-desktop /usr/share/applications/telegramdesktop.desktop
telegram-desktop /usr/share/icons/
telegram-desktop /usr/share/icons/hicolor/
telegram-desktop /usr/share/icons/hicolor/128x128/
telegram-desktop /usr/share/icons/hicolor/128x128/apps/
telegram-desktop /usr/share/icons/hicolor/128x128/apps/telegram.png
telegram-desktop /usr/share/icons/hicolor/16x16/
telegram-desktop /usr/share/icons/hicolor/16x16/apps/
telegram-desktop /usr/share/icons/hicolor/16x16/apps/telegram.png
telegram-desktop /usr/share/icons/hicolor/256x256/
telegram-desktop /usr/share/icons/hicolor/256x256/apps/
telegram-desktop /usr/share/icons/hicolor/256x256/apps/telegram.png
telegram-desktop /usr/share/icons/hicolor/32x32/
telegram-desktop /usr/share/icons/hicolor/32x32/apps/
telegram-desktop /usr/share/icons/hicolor/32x32/apps/telegram.png
telegram-desktop /usr/share/icons/hicolor/48x48/
telegram-desktop /usr/share/icons/hicolor/48x48/apps/
telegram-desktop /usr/share/icons/hicolor/48x48/apps/telegram.png
telegram-desktop /usr/share/icons/hicolor/512x512/
telegram-desktop /usr/share/icons/hicolor/512x512/apps/
telegram-desktop /usr/share/icons/hicolor/512x512/apps/telegram.png
telegram-desktop /usr/share/icons/hicolor/64x64/
telegram-desktop /usr/share/icons/hicolor/64x64/apps/
telegram-desktop /usr/share/icons/hicolor/64x64/apps/telegram.png
telegram-desktop /usr/share/metainfo/
telegram-desktop /usr/share/metainfo/telegramdesktop.appdata.xml
```

Or the other way around, I can find out what package ows a specific file:

```
$ pacman -Qo /usr/bin/telegram-desktop
/usr/bin/telegram-desktop is owned by telegram-desktop 3.1.9-1
```

That is, just a quick article on searching for packages. This is definitely something that will come up sooner or later when working with Linux, so it's useful to know that pretty early on.
