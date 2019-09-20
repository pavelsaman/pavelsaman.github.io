---
layout: page
title:  "Change LightDM GTK Greeter Background with Bash and Cron"
date:   2019-09-19 22:07:19 +0200
categories: linux
---

When using LightDM display manager, one also has to install a greeter. I’ve been using LightDM GTK Greeter, which is a light greeter that could be configured in a few ways. However, one of the things I want to change regularly is the greeter background, so I get a new picture on my login screen every week.

When I have a look into LightDM GTK settings, I can see I can easily change its background: ![image](/images/lightdm_greeter_settings.png)

On the other hand, I have more than just one picture I like and I'd like to rotate them every now and then, let's say once a week.

The way I approached this is a small bash script that I'm now going to share in this post.

First off, we need to find out where is a config file of the lightdm greeter. I'd try `man lightdm` and scroll down to the FILES section:

```bash
FILES
       /etc/lightdm/lightdm.conf
              Configuration

       /etc/lightdm/users.conf
              User list configuration (if not using Accounts Service)

       /etc/lightdm/keys.conf
              XDMCP keys
```

I don't really see anything related to lightdm greeter, but directory `/etc/lightdm/` looks promising.

```bash
$ ls -l /etc/lightdm/
total 28
-rw-r--r-- 1 root root   40 May 17 16:59 keys.conf
-rw-r--r-- 1 root root 7061 Sep  7 21:39 lightdm.conf
-rw-r--r-- 1 root root  468 Sep 19 21:49 lightdm-gtk-greeter.conf
-rw-r--r-- 1 root root  477 Sep 19 21:49 lightdm-gtk-greeter.conf.bak
-rw-r--r-- 1 root root  465 May 17 16:59 users.conf
-rwxr-xr-x 1 root root 1439 May 17 16:59 Xsession
```

It seems I'm getting somewhere here since I see file `lightdm-gtk-greeter.conf`, that's exactly what I need. Let's have a look inside:

```bash
$ cat /etc/lightdm/lightdm-gtk-greeter.conf.bak 
[greeter]
background = /usr/share/backgrounds/illyria-default-lockscreen.jpg
font-name = Cantarell Bold 12
xft-antialias = true
icon-theme-name = Adapta-Papirus-Maia
screensaver-timeout = 60
theme-name = Adapta-Eta-Maia
cursor-theme-name = xcursor-breeze
show-clock = false
default-user-image = #avatar-default
xft-hintstyle = hintfull
position = 50%,center 50%,center
clock-format = %d/%m/%Y, %H:%M:%S
panel-position = bottom
indicators = ~host;~spacer;~clock;~spacer;~power
```

I've already changed the default setting a bit, so you might see a different content. However, the interesting part is `background = `. In order to change the background, I need to change this line, and do it regularly. But let's take is step by step and find out first what I need:
1. a backup would be nice if I happen to break/delete the file, therefore I'll create `/etc/lightdm/lightdm-gtk-greeter.conf.bak` (you can already see this above)
2. I need to change `background = ` line in `/etc/lightdm/lightdm-gtk-greeter.conf`. I'll go for bash and some text processing tool for changing what I said in 1.
3. I need a directory with pictures I want to use for the rotation
4. I need to run 1. regularly, say once a week; I'll do this in cron

When it comes down to the bash script, I came up with the following one:

```bash
#!/usr/bin/env bash

dir="/usr/share/backgrounds/greeter"
# file to change
config_file="/etc/lightdm/lightdm-gtk-greeter.conf"
# alwas hae a backup
bak_file="/etc/lightdm/lightdm-gtk-greeter.conf.bak"

# get all background pic
function get_all_pictures {
	all_pictures=(${dir}/*)
}

# choose one
function choose_random {
	new_pic_index=$((0 + RANDOM % ${#all_pictures[@]}))
}

# change the line
function change {
	awk -F'=' -v NEW_PIC=${all_pictures[${new_pic_index}]} '{ if ($1 == "background ") {print "background = "NEW_PIC} else {print} }' "${bak_file}" > "${config_file}" 
}

get_all_pictures || exit 1
choose_random || exit 1
change && exit 0 || exit 1
```

As you can see, I first define some variables for the config file as well as for the backup file that's used here for reading from it. I've also chosen a particular directory where I put all my pictures I want to rotate, they will go inside `/usr/share/backgrounds/greeter`. Then I just get names of all pictures inside the directory (function `get_all_pictures`) and choose one at random (function `choose_random`). If any of these functions fail from whatever reason, I terminate the script with exit code 1. The last part is running an awk script where I read lines from the config file and change the one starting with `background = `.

The last part is to set up a crontab, in my case, it looks like this:

```bash
$ sudo crontab -l
# change greeter background every Saturday
0 19 * * 6 /usr/bin/change_greeter_background.bash
```

That line means I change it every Saturday at 7 p.m.

You can also get the script from [this repository](https://github.com/pavelsaman/BashWorkspace.git).
