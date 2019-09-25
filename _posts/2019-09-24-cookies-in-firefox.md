---
layout: page
title:  "Cookies in Firefox"
date:   2019-09-24 09:07:19 +0200
categories: browser linux
---

This is just a quick tip on how to see all cookies stored in Firefox.

On Linux, Firefox creates a directory in `~/.mozilla`, inside that there are more stuff, but in terms of cookies, I need to go into a directory dedicated to my current profile, so sth like `~/.mozilla/firefox/av1831zo.default`.

In here, there are two files:

```bash
$ ll | grep cookie
-rw-r--r--  1 pavel pavel 1.0M Sep 24 18:13 cookies.sqlite
-rw-r--r--  1 pavel pavel 577K Sep 24 18:14 cookies.sqlite-wal
```

Firefox uses sqlite database, which is only one file, but it could be used as a simple database, e.g. I can query it using SQL commands. For that I use `sqlitebrowser`:

```bash
$ pacman -Qs sqlitebrowser
local/sqlitebrowser 3.11.2-1
    SQLite Database browser is a light GUI editor for SQLite databases, built on top of Qt
```

If your Firefox is open, I don't really succeed in opening the file `cookies.sqlite`, it's locked:

![image](/images/cookies_locked.png)

In this case, just copy the file elsewhere or close Firefox.

After opening the file in `sqlitebrowser`, I can query it e.g. all secure cookies currently stored in my Firefox profile:

```sql
select * from moz_cookies t where t.isSecure = 1;
```

![image](/images/sqlitebrowser.png)