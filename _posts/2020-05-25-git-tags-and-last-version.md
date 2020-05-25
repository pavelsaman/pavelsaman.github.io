---
layout: page
title:  "Git Tags and Last Version"
date:   2020-05-25 18:50:19 +0200
categories: other
---

One of the features git supports is tags (see `$ man git-tag`), which is another useful bit that can help you find more information quickly. Let's have a look at some basics, and what I use these tags for.

Every revision in git is assigned a unique SHA-1 hash, that's often enough to identify a concrete point in the history. But it's not very human-readable, so there's another feature that allows for creating names/identifiers that are associated with certain revisions.

I can see tags in `git log`:

```
$ git log --tags
```

which allows me to see all commits from the newest tag back:

```
8ddb026 (tag: 2.14.0, origin/master, master) Merge branch 'release/2.14.0'
...
```

I can also type:

```
$ git tag
```

and see all tags as a simple list:

```
1.0.3-job
1.0.3-web+jobs
2.10.0
2.11.0
2.11.1
2.11.2
2.12.0
2.12.1
2.13.0
2.13.1
2.14.0
2.5.0
2.5.1
2.6.0
2.7.0
2.7.1
2.7.2
2.7.3
2.8.0
2.8.1
2.9.0
job
job-1.0.2
release/1.2.0
release/2.0.1
release/2.0.2
release/2.0.3
release/2.0.5-web-cz
release/2.0.6-web-jobs
release/2.0.7-web-jobs
release/2.1.0-web-jobs
release/2.1.4-web-jobs
release/2.2.0
release/2.3.0
release/2.3.1
web-1.0.3
```

or I can print them in columns:

```
$ git tag --column
1.0.3-job               2.12.0                  2.5.1                   2.8.0                   release/2.0.1           release/2.1.0-web-jobs
1.0.3-web+jobs          2.12.1                  2.6.0                   2.8.1                   release/2.0.2           release/2.1.4-web-jobs
2.10.0                  2.13.0                  2.7.0                   2.9.0                   release/2.0.3           release/2.2.0
2.11.0                  2.13.1                  2.7.1                   job                     release/2.0.5-web-cz    release/2.3.0
2.11.1                  2.14.0                  2.7.2                   job-1.0.2               release/2.0.6-web-jobs  release/2.3.1
2.11.2                  2.5.0                   2.7.3                   release/1.2.0           release/2.0.7-web-jobs  web-1.0.3
```

The idea behind tags is that they provide another way to mark certain commits. As it's visible in my examples, tags are usually used for application version, e.g. when we finished version 2.14.0, we tagged the last commit of this version.

In my daily work life, I often need to know the last version I work with, or the last version released. I can either go to Azure DevOps and keep clicking for a bit and find it there, or I can use this very command line interface and find it out much faster.

As it turns out, all I need is to extract and sort the output of `git tag` and return the first line. Alright, that should be easy in bash:

```
$ git tag | egrep "^[0-9]+[.][0-9]+[.][0-9]+$" | sort -t'.' -k1,1 -k2,2 -k 3,3 -h -r | head -1
```

It has some limitations, it filters out all tags that do not consist of only numbers separated by dots. It's not actually a limitation for me right now (those with non-numeric characters are not really used in our project anymore), but it might be for you if your tags look differently, so be aware of copy-pasting :)

This is also quite long and not anything I want to type every time I want to find out the last tag (it should save me some time as I mentioned). Therefore, I'll create a bash function I source in `.bashrc`:

```bash
lasttag() {
    if [[ $(g) =~ ^Y|y$ ]]; then
        git tag | egrep "^[0-9]+[.][0-9]+[.][0-9]+$" | sort -t'.' -k1,1 -k2,2 -k 3,3 -h -r | head -1
    else
        printf "%s\n" "Not a git repository"
    fi
}
```

If you're wondering what that `g` function is, that's another custom function I created. It simply returns `Y` or `n` based on whether or not the current dir contains a local git repo. Another little simplification of mine.

If I source my custom bash function (yes, I keep them separately):

```bash
if [[ -f ~/.bash_functions ]]; then
    . ~/.bash_functions
fi
```

I can later use just:

```
$ lasttag 
2.14.0
```

in order to get the last tag that represents the last version of the app.

Quite a tiny and simple thing to do but it can save a few minutes over a working week.