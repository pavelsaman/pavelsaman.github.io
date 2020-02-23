---
layout: page
title:  "Fancy git log"
date:   2020-02-23 16:07:19 +0200
categories: other
---

[Git log](https://git-scm.com/docs/git-log) command offers a variety of things to set up, it can be overwhelming to read the whole man page and to use it effectively when you just want to print out some commit history. I've recently read up on git log a bit more, so I can share how my git log outcome looks like at the moment.

For many months, I was accustomed to using `git log --oneline`, which was good enough, but it also drove me mad at times (like having to press q for quitting when there're too many lines of outcome etc.). So the only option was to read a bit more about how I can tweak the outcome of git log.

First of all, there're a few requirements I have for my git log:

- it has to be simple, because I mostly use it just for checking a few last commits and the commit messages
- it has to include time, name, email, commit hash (abbreviated is better than full)
- it has to include commit messages
- it should be somehow formatted, so I can easily and quickly read it

The result is the following git log:

```
git log --pretty=format:"%h%Cred%d%Creset %Cgreen%aI%Creset - %an - [%ae]%n %s%n" -n10
```

Which gives me an outcome like this:

```
a671e6d (HEAD -> dev, origin/dev) 2020-02-22T20:13:49+01:00 - Pavel Saman - [pavel.saman@test.cz]
 Admin.Tests - GetOrderProductVariants - new tests for order

c13d528 2020-02-22T17:25:28+01:00 - Pavel Saman - [pavel.saman@test.cz]
 Admin.Tests - more tests from product endpoints

...
```

Simple enough for getting a quick info about the commit history. I've aliased this command, so it can be quickly utilized.

