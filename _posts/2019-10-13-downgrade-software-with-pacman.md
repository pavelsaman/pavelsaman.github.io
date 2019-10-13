---
layout: page
title:  "Downgrade Software with Pacman"
date:   2019-10-13 15:07:19 +0200
categories: linux
---

It's certainly more common to upgrade software, but on those rare occasions when something breaks with a higher version, it's useful to know how to downgrade a particular package.

I've recently upgraded Eclipse and RED. I use these for writing tests in Robot framework. However, I got stuck on [this error](https://github.com/nokia/RED/issues/324). After searching for a while and even asking on [stackexchange](https://sqa.stackexchange.com/q/41124/41012), it was pretty sure I had to downgrade Eclipse.

Since I had this issue on a Windows machine, I didn't bother much and just installed one more version of Eclipse. It's a solution as well, not as neat as on Linux, though.

However, I want to share these steps for downgrading a package on Linux.

First, I checked if I still have an older version in cache:

```bash
$ grep -i cachedir pacman.conf
$ ll /var/cache/pacman/pkg/
```

Sure enough, there was an older version of Eclipse. So let's downgrade:

```bash
$ sudo pacman -U /var/cache/pacman/pkg/eclipse-java-4.12-1-x86_64.pkg.tar.xz
$ sudo pacman -U /var/cache/pacman/pkg/eclipse-common-4.12-1-x86_64.pkg.tar.xz
```

Done :) Fairly easy. There's, however, one more step I advise you take. If you perform upgrade of your system right now, Eclipse would again be upgraded. But that's not exactly what I want since the bug is not solved yet. Fortunately enough, I can change `/etc/pacman.conf` so some packages won't be upgraded:

```bash
$ grep -i eclipse pacman.conf
IgnorePkg   = eclipse-common eclipse-java
```

Any time I use `pacman`, it will ignore these two packages. It will also warn me about the fact I choose to ignore their upgrade:

```bash
$ sudo pacman -Syu
...
warning: eclipse-common: ignoring package upgrade (4.12-1 => 4.13-1)
warning: eclipse-java: ignoring package upgrade (4.12-1 => 4.13-1)
...
```

Nice and easy. It's also pretty fast, no need to install a new package, uninstall the faulty one etc.
