---
layout: page
title:  "Resolving Common Problems With R Package Installation"
date:   2020-08-08 19:30 +0200
categories: linux
---

I've recently started playing around with R, which also translates into installing additional R packages. Right at the beginning, I ran into two problems which I'll explain here now.

I'm not using any conventional linux distibution like Red Hat, Ubuntu, openSUSE etc. so some installation tasks might take a bit longer because I need to resolve some dependencies etc. I don't know if this was the case here or it's just the way I use my system.

### tcl/tk

The very first error I came across was this one:

```
> install.packages("RMySQL")
Installing package into ‘/home/pavel/R/x86_64-pc-linux-gnu-library/4.0’
(as ‘lib’ is unspecified)
--- Please select a CRAN mirror for use in this session ---
Error: .onLoad failed in loadNamespace() for 'tcltk', details:
  call: dyn.load(file, DLLpath = DLLpath, ...)
  error: unable to load shared object '/usr/lib/R/library/tcltk/libs/tcltk.so':
  libtk8.6.so: cannot open shared object file: No such file or directory
```

R uses tcl/tk, but that was not present on my system. Fortunately, the solution here was straighforward:

```
# pacman -Su tk
```

Another solution could be to disable menus as described [here](https://stackoverflow.com/questions/7430452/disable-suppress-tcltk-popup-for-cran-mirror-selection-in-r), that is by setting:

```
> options(menu.graphics=FALSE)
```

Yet another solution could be to specify a mirror right when installing a package, that is not to wait for the tcl/tk popup to open:

```
> install.packages("RMySQL", repos='https://mirrors.nic.cz/R/')
```

The list of repos could be found on [CRAN](https://cran.r-project.org/mirrors.html).

### `/tmp` and execution permission

I'm smarter now, so I can start by quoting the [official admin documentation](https://cran.r-project.org/doc/manuals/r-release/R-admin.html):

> and /tmp exists and can be written in and scripts can be executed from

But this is not always true on all systems. Especially if you have set up `noexec` in `/etc/fstab` like me:

```
tmpfs                                     /tmp           tmpfs   rw,size=3G,noexec,nodev,nosuid,nouser,noatime,auto,async,mode=1700 0 0
```

If set like so, then when installing some packages, the following error will pop up:

```
* DONE (bit64)
* installing *source* package ‘odbc’ ...
** package ‘odbc’ successfully unpacked and MD5 sums checked
** using staged installation
ERROR: 'configure' exists but is not executable -- see the 'R Installation and Administration Manual'
* removing ‘/home/pavel/R/x86_64-pc-linux-gnu-library/4.0/odbc’

The downloaded source packages are in
    ‘/tmp/Rtmp7gPtYt/downloaded_packages’
Warning message:
In install.packages("odbc") :
  installation of package ‘odbc’ had non-zero exit status
```

which I didn't know what to do with until I found that line in the official documentation.

You can verify what tmp dir R sees by typing:

```
> tempdir()
[1] "/tmp/Rtmp7gPtYt"
```

One solution is obvious, that is changing the `noexec` in `/etc/fstab`. I didn't like this, so I eventually went for the following solution:

```
$ mkdir ~/tmp
$ export TMPDIR=~/tmp
```

then you need to restart R and the next time R will use this directory as a temp one. And the error disappears.

