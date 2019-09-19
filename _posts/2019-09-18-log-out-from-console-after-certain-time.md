---
layout: page
title:  "Log Out From Console after Certain Time"
date:   2019-09-18 17:00:19 +0200
categories: linux
---

It is possible to forget to log out from a console, which might be a security risk. It would be nice to have a way to log out automatically if this happens.

### `/etc/profile` and `/etc/profile.d/`

As it turns out, we can do such a thing with just a few lines of shell code. Let's write this:

```sh
TMOUT=30

case $( /usr/bin/tty ) in
	/dev/tty[0-9]*)
		export TMOUT
		;;
esac
```

Variable `TMOUT` could be used to log out a user after a specified number of seconds. If set to `30` like in my example, it'll log out a user after 30 seconds of innactivity.

The only question now is where to put such a script?

Let's have a look at `/etc/profile`:

```sh
# /etc/profile

# Set our umask
umask 022

# Append our default paths
appendpath () {
    case ":$PATH:" in
        *:"$1":*)
            ;;
        *)
            PATH="${PATH:+$PATH:}$1"
    esac
}

appendpath '/usr/local/sbin'
appendpath '/usr/local/bin'
appendpath '/usr/bin'
unset appendpath

export PATH

# Load profiles from /etc/profile.d
if test -d /etc/profile.d/; then
	for profile in /etc/profile.d/*.sh; do
		test -r "$profile" && . "$profile"
	done
	unset profile
fi

# Source global bash config
if test "$PS1" && test "$BASH" && test -z ${POSIXLY_CORRECT+x} && test -r /etc/bash.bashrc; then
	. /etc/bash.bashrc
fi

# Termcap is outdated, old, and crusty, kill it.
unset TERMCAP

# Man is much better than us at figuring this out
unset MANPATH
```

This is a standard content of the file on Manjaro Linux. This file gets read by bash when it's invoked as a login shell, so it's a good place to set up some global config for all users (just like here setting `PATH` and `umask` for example).

One particular piece I'm interested in is:

```sh
# Load profiles from /etc/profile.d
if test -d /etc/profile.d/; then
	for profile in /etc/profile.d/*.sh; do
		test -r "$profile" && . "$profile"
	done
	unset profile
fi
```

This part works with every file in `/etc/profile.d/` that ends with `.sh`. If such a file exists and read permissions are granted, it will execute such a file. That's exactly what we need for our tiny script from the beginning.

Let's save our script as `shell-timeout.sh`:
```sh
ls -l shell-timeout.sh 
-rw-r--r-- 1 root root 77 Sep 17 16:12 shell-timeout.sh
```

Now it's a good time to test it. Log into a console (tty, not pts) and just wait for 30 seconds. After the time period, the user should be logged out automatically.
