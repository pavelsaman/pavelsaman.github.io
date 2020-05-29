---
layout: page
title:  "Git Aliases"
date:   2020-05-29 20:50:19 +0200
categories: other
---

Git offers aliases for its commands, which is a nice way to make some commands shorter or more memorable. Let's see how to create and delete such git aliases.

First of all, git aliases could be defined globally or only for a repository. If I want to see my global setting, I'd type:

```
$ git config --global -l
alias.unstage=restore --staged --
alias.line=log --oneline
alias.graph=log --oneline --graph
alias.merged=branch --merged
alias.unmerged=branch --no-merged
```

This setting is also stored in a file that stores global configuration, it usually is `~/.gitconfig`:

```
$ cat ~/.gitconfig 
[alias]
	unstage = restore --staged --
	line = log --oneline
	graph = log --oneline --graph
	merged = branch --merged
	unmerged = branch --no-merged
```

Another way is to set aliases only for a particular repository. Then such aliases won't show up in the previous command with `--global` option and won't be stoed in `~/.gitconfig`.

As you can see, I already use some aliases. But let's create a new one, I'll make it a global alias:

```
$ git config --global alias.l 'log'
```

Now I can use `$ git l` to get the same result as when I use `$ git log`. If I want to define an alias for only one repository, I'd do the following:

```
$ cd my-repo
$ git config alias.d 'diff'
```

The alias will be stored in `my-repo/.git/config`, not in the global configuration file.

When I know how to create aliases, it might be useful to know how to delete them as well:

```
$ git config --unset alias.l
```

If I want to delete a global alias, I'd simply add `--global` option to the previous command:

```
$ git config --global --unset alias.l
```

Another way is to delete certain lines in git configuration files.

This is a simple feature that git offers, but it can make your time with git much more enjoyable when you don't need to remember certain long commands with a few options on top of that.