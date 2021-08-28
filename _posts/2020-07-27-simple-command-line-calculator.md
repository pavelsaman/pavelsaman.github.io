---
layout: page
title:  "Simple Command Line Calculator"
date:   2020-07-27 19:15:19 +0200
categories: linux
---

From time to time there's a need for simple math calculations. You can use a variety of different calculators, usually graphical ones. I'll take a different approach and show one (out of many) way of doing math in the command line.

If you use bash as the main shell, the first idea might be to use the interpreter to do some math:

```bash
$ echo $((5+6))
11
```

That's fine as long as you use integers. If you try something like:

```bash
$ echo $((5+6.32))
bash: 5+6.32: syntax error: invalid arithmetic operator (error token is ".32")
```

bash will complain. There's absolutely nothing wrong with your syntax, it's such that bash was never created for this purpose, and so it does not support float numbers. If you still want to use a shell for calculations, pick another one like zsh.

However, I think many people are so used to bash that there's simply no chance of switching to something else. In this case, you have roughly four options:

- you just use your head and no calculator
- you open some GUI calculator
- you install another package
- you use tools already installed

Writing about each of these would take too long, so I'll focus on the last one, specifically on using `bc` for this purpose.

First off, the man page says:

```
NAME
       bc - An arbitrary precision calculator language
```

That seems promissing.

If I need bc to only perform some basic additions, etc., I can dive straight in and experiment a bit.

```bash
$ echo "2+2" | bc
4
```

Looks good. How about float numbers:

```bash
$ echo "2+2.2" | bc
4.2
```

Perfect. I just solved the limitation of bash.

It'd be nice to shorten this command a bit, let's use a here string:

```bash
$ bc <<<"2+2.8"
4.8
```

Much better, But I still need to type `<` three times, which takes time. How about creating a function out of this?

```bash
calc() {
    bc <<<"$1"
}
```

I'll put it inside `/etc/bash.functions`, so it will be available to whoever source the file.

Then I can go and use it simply like this:

```bash
$ calc 5+9.87
14.87
```

At this point, I'm quite happy with it, there's definitely some improvement from the first version with `echo` :)

The bottom line of this could be that many times you don't need to install other packages, look for what's already available on your system. There's often more than what you expect.
