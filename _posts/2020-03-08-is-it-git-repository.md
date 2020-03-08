---
layout: page
title:  "Is it a git repository?"
date:   2020-03-08 16:07:19 +0200
categories: linux
---

I like to use simple shortcuts that save just a couple of characters that I'd otherwise need to type out. One such abbreviation I created for myself is the following one for finding out whether or not I'm currently in a git repository.

Following on the [example of function `ex()` from `.bashrc` on Manjaro](https://gist.github.com/jlhg/9366374#file-manjaro-bashrc-sh-L65), I created a simple function `g()` that looks as follows:

```
g() {
	[[ -d ".git" ]] && printf '%s\n' 'Y' || printf '%s\n' 'n'
}
```

I'll simply print `n` or `Y` depending on whether or not it finds `.git` directory in the current directory.

I admit this is not a completely universal solution because just a presence of `.git` doesn't automatically mean it's a git repository, but this is what I'm happy to live with in my particular situation. It still saves me a few hits on my keaboard once in a while.
