---
layout: page
title: "Locking Linux User Accounts"
date: 2021-10-31 12:30 +0200
categories: linux
---

Let's review some of the ways how a user can be locked on a Linux system.

First of all, locking refers to user passwords, so technically you lock the password. It's a different concept from setting a user account as expired. For example: you can lock a user password, but the user can still log in if they use a different authentication method (ssh keys, some HW like Yubikey, etc.).

Ok, then how to go about locking a user password?

There are various utilities that can do it, or it could be done by editing `/etc/shadow` manually (not recommended, it's easy to make a mistake and mess things up).

The thing is that locking a user password really does only one thing, which is it prepends `!` to a password hash of the user. The hash is usually stored in `/etc/shadow`:

```
$ sudo grep pavel-remote /etc/shadow
Please touch the device.
[sudo] password for pavel: 
pavel-remote:$6$JF5a16FxyxL6ZhWe$UYsvSGYfzjhxySUBth3lAHAg7u4lfntE9nwhsHRPEAq7lB7XdMDdZ9dtBlgwOQZZIMgCJketOdYUmJibxqrXC.:18931:0:99999:7:::
```

The second field is a password hash, more precisely salted SHA-512 (notice `$6`). A locked password will look like:

```
$ sudo grep pavel-remote /etc/shadow
Please touch the device.
[sudo] password for pavel: 
pavel-remote:!$6$JF5a16FxyxL6ZhWe$UYsvSGYfzjhxySUBth3lAHAg7u4lfntE9nwhsHRPEAq7lB7XdMDdZ9dtBlgwOQZZIMgCJketOdYUmJibxqrXC.:18931:0:99999:7:::
```

So in theory you can edit the file. But such manual changes are error-prone, so it's much better to use some utility like `passwd` or `usermod`.

### passwd

With `passwd`, I can lock a user password like so:

```
$ sudo passwd -l pavel-remote
```

And to unlock a user password:

```
$ sudo passwd -u pavel-remote
```

### usermod

With `usermod`, it's very similar, just with capital L and U:

```
$ sudo usermod -L pavel-remote
```

And to unlock:

```
$ sudo usermod -U pavel-remote
```

I can type out a simple shell script that will report on locked accounts:

```sh
#!/bin/sh

[ ! -r /etc/shadow ] && { echo "No read permission on /etc/shadow" 1>&2; exit 1; }

locked_accounts=$(grep '^.*:!' /etc/shadow | cut -d: -f1 | tr '\n' ' ')
echo $locked_accounts

exit 0
```

Obviously this script has to be run with root permissions because `/etc/shadow` is owned by `root:root` and is not world readable.

Last but not least, both man pages for `passwd` and `usermod` warn about the difference between locking a user password and locking a user account (expiration), so it's worth reading the man pages :)
