---
layout: page
title:  "Upgrading Software in Windows and Linux"
date:   2020-01-31 16:07:19 +0200
categories: linux
---

I've been a regular user of both Windows and Linux platforms. Only after a little while, I started to appreciate the ease with which I can upgrade software and/or the system itself in Linux. In this article, I'd like to have a look at this very thing from the users's perspective.

## Updating Software in Windows

The stadard way to update software on Windows machines is to go into each one of your programs, look for some "find-updates" button (which is usually well hidden somewhere in menu), press that, wait a while and see whether or not there's a new version available. My guess is there might be at least 10 or so programs that almost anybody uses on a regular basis, and so this very process of finding out updates can easily take up about 15 minutes in total.

Another way is to set up your software in a way that it will look for updates on its own. If new updates are available, it might show you a notification or even update itself. Both of these options are not what I want, the former because notifications drive me away from work, the latter because I want to have a bit more control over new versions.

All in all, it's never completely easy and straighforward.

## Package Managers in Linux

Another platform I know my way around is Linux - a few Debian based distributions, and Manjaro. The way to find out possible new versions of software is one command `$ sudo pacman -Sy` or `$ sudo pacman -Syy` to force a refresh of the package database. I could be further piped into `$ pacman -Qu` so I can easily see what updates are available for all of my packages and the system. It takes about 5 seconds to run these commands and see the result.

If `apt` is available, it's again very easy and straighforward, first update the package database `$ apt update` and then see what updates are available `$ apt list --upgradable`. Again, it takes about 5 seconds.


I've also heard an argument that it's so complicated in Linux. Well, I don't think that issuing about 3 commands is such a pain. And if it is, there're graphical wrappers around these low level package managers, so no need to work in the console/terminal, it could be done via GUI as well.
