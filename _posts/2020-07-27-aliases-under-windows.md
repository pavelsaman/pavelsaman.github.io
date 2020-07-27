---
layout: page
title:  "Aliases Under Windows"
date:   2020-07-27 14:30:19 +0200
categories: windows
---

Windows cmd envrionment is hard to navigate. One thing that makes it inconvenient is the absence of a straightforward way of defining aliases. Let's have a look at one solution.

I'd say at the beginning that there are more ways of doing this. I'll share one that I currently use for about 15 or so aliases that I've defined for myself. If your needs are different, e.g. you need much more aliases, you might go for a different solution.

Let's assume I want to use `ls -la --color` just as `ll`. The way I'd do that is as follows.

I'd set up a new folder right under `C:\` and name it `Aliases`. This folder will contain one batch file per one alias. The name of each batch file will contain the alias itself plus of course the `.bat` extension. So in my example I'd create a `ll.bat` file inside `C:\Aliases\` folder.

Then the content of the batch file will be simple in this case:

```
@echo off
ls -la --color
```

Last thing I need to do is to add `C:\Aliases\` into `PATH` so the folder is searched when I type `ll` into the command line.

Another example could be when I want to use `gc ...` for `git checkout ...`. The only thing I need to take into account here is how to allow for additional parameters to be passed from the command line into the batch file command. It solution could be:

```
@echo off
git checkout %*
```

Then I can go and type:

```
>gc master
Switched to branch 'master'
Your branch is up to date with 'origin/master'.
```

And it works as expected.

I've created a [repository](https://github.com/pavelsaman/WindowsCmdAliases) on my Github page where you can get more aliases I currently use.

Windows command line environment is not the most usable one I know. But there're still ways of making it a bit more user-friendly and not as time consuming. Defining aliases this way is one way of doing so.