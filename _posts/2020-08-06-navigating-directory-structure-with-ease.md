---
layout: page
title:  "Navigating Directory Structure With Ease"
date:   2020-08-06 22:30 +0200
categories: linux
---

Changing directories is perhaps one of the most common tasks one needs to do. If this is true, then every letter counts and it's useful to think about ways of making it even easier. I'll show a few `cd` tips and tricks I use.

### cd aliases

Of course, cd is already brief enough, yet if you need to type it often, perhaps every single letter makes a difference. There're a couple of aliases I use:

```
alias cd-="cd -" # toggle
alias ..="cd .."
alias ...="cd ../.."
alias ....="cd ../../.."
alias .....="cd ../../../.."
```

I've experimented with more options, I've used `cd..` for going one level up, now I've settled for these. Two dots to go one level up and then one dot for every additional level. I rarely use more than two levels up though.

Some other similar examples could be:

```
alias cd..="cd .."
```

or using numbers:

```
alias ..2="cd ../.."
alias ..3="cd ../../.."
```

### cd with no arguments

If cd is used without arguments and `$HOME` is set to a non-empty value, it will behave as if you executed `cd $HOME`. You surely read man pages, right?

```
2. If no directory operand is given and the HOME environment variable is set to a non-empty value, the cd utility shall behave as if the directory named
   in the HOME environment variable was specified as the directory operand.
```

So going to my home directory is just a matter of typing `$ cd`.

### toggle between last two directories

`$ cd -` is a way to toggle between last two visited directories. I also aliases this command to `$ cd-`, one letter shorter :)

### $CDPATH

Setting `$CDPATH` is more like setting `$PATH` except the former is used only for the cd command. If I often use `/etc` and its subdirectories, I can set up

```
export CDPATH=/etc
```

then if I type `$ cd iptables`, I will be taken into `/etc/iptables`. No need to specify the absolute path, create alias, or go first to `/etc`.

`$CDPATH` could also contain more directories, very much like `$PATH`.

### mkdircd

When I create a new directory, I often want to go in it. But `mkdir` is only for directory creation. Sure I can write `$ mkdir -p test && cd test`, but that's already too much typing. And any attempt at some bash script like:

```bash
#!/usr/bin/bash

mkdir -p "$1" && cd "$1"
```

will prove fruitless since such a script will be executed as a different process, so it will have its own environment and when the script exits, nothing will be taken over by the shell that executed the script.

But there a solution. I created the following bash function that created a directory and goes in it:

```bash
# mkdir && cd
mkdircd() {
    mkdir -p "$1" && eval cd "\"\$$#\"";
}
```

Perhaps there's more, directory stack with `pushd`, `popd`, and `dirs` is another example, but this is all I want at present. Short tips that save me some time and typing :)