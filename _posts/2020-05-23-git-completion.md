---
layout: page
title:  "Git Completion"
date:   2020-05-23 17:30:19 +0200
categories: other
---

Git offers so many commands and options that it's virtually impossible to remember them all, especially when one doesn't work with them daily. However, git also offers automatic completion that it perhaps not very knows but nevertheless very useful.

Let's assume I want to see information about a particular remote repository. If I remember the command, I can type:

```
$ git remote show covid
* remote covid
  Fetch URL: https://github.com/CSSEGISandData/COVID-19.git
  Push  URL: https://github.com/CSSEGISandData/COVID-19.git
  HEAD branch: master
  Remote branches:
    CSSEGISandData-patch-02042020 tracked
    master                        tracked
    web-data                      tracked
  Local branches configured for 'git pull':
    master   merges with remote master
    web-data merges with remote web-data
  Local refs configured for 'git push':
    master   pushes to master   (up to date)
    web-data pushes to web-data (up to date)
```

But if I don't remember what goes after remote?

What almost anybody who has ever worked with a terminal (or any at least a bit advanced text interface) does in this situation is to hit `TAB` twice in the hope that the started command sequence will be taken into account and the user will be presented with a list of commands that could follow.

But more often than not that's not the case with a default git configuration, the user simply gets a list of files in the current directory:

```
$ git remote [TAB][TAB]
archived_data/                  .git/                           README.md                       
csse_covid_19_data/             .gitignore                      who_covid_19_situation_reports/
```

This is not what I want, it does not help me complete the command. What is the solution then?

Fortunately enough, this completion functionality is actually provided by the creators of git. You can browse to the official [git repository on Github](https://github.com/git/git), get the source code, and navigate to `contrib/completion` directory:

```
$ mkdir git-source
$ cd git-source
$ git clone https://github.com/git/git.git
...
$ cd git/contrib/completion/
```

And lo and behold you can see here a few interesting scripts:

```
$ ls -l
total 108
-rw-r--r-- 1 pavel pavel 73075 May 23 17:05 git-completion.bash
-rw-r--r-- 1 pavel pavel  4801 May 23 16:59 git-completion.tcsh
-rw-r--r-- 1 pavel pavel  6124 May 23 16:59 git-completion.zsh
-rw-r--r-- 1 pavel pavel 16858 May 23 16:59 git-prompt.sh
```

Based on your shell you choose one and place it in some sensible location (home directory is a good choice, or something like `~/.local/bin`). Then you source the script in `.bashrc` (or in the global version of it if you want to allow this feature for all users):

```bash
# git completion if exists
if [[ -f ~/.git-completion.bash ]]; then
    . ~/.git-completion.bash
fi
```

Technically, you don't need that extra condition. On the other hand, if you ever (by mistake) delete the file, you won't be presented with the following annoying message when you open a new terminal window:

```
bash: /home/pavel/.git-completion.bash: No such file or directory
```

So I usually make this little effort and source such scripts inside a condition.

Back to the first step. When I now type a part of a git command and then press `[TAB]` twice, I get a list of commands that could follow:

```
$ git remote [TAB][TAB]
add            get-url        prune          remove         rename         set-branches   set-head       set-url        show           update
```

How useful!

It's even mroe useful for options. `git log` has more than 100 of them, that's too many to remember, but with completion, there's no need to remember them:

```
$ git log --[TAB][TAB]
Display all 121 possibilities? (y or n)
--abbrev                   --date=                    --full-diff                --minimal                  --no-textconv              --simplify-merges 
--abbrev=                  --date-order               --full-history             --min-parents=             --no-walk                  --since=
--abbrev-commit            --decorate                 --full-index               --name-only                --no-walk=                 --sparse 
--after=                   --decorate=                --graph                    --name-status              --numstat                  --src-prefix=
--all                      --dense                    --grep=                    --no-abbrev-commit         --oneline                  --stat 
--all-match                --diff-algorithm=          --histogram                --no-color                 --parents                  --submodule 
--author=                  --diff-filter=             --ignore-all-space         --no-color-moved           --patch                    --submodule=
--before=                  --dirstat                  --ignore-blank-lines       --no-color-moved-ws        --patch-with-stat          --summary 
--binary                   --dirstat=                 --ignore-cr-at-eol         --no-decorate              --patience                 --tags 
--branches                 --dirstat-by-file          --ignore-space-at-eol      --no-expand-tabs           --pickaxe-all              --text 
--check                    --dirstat-by-file=         --ignore-space-change      --no-ext-diff              --pickaxe-regex            --textconv 
--cherry-mark              --do-walk                  --ignore-submodules        --no-indent-heuristic      --pretty=                  --topo-order 
--cherry-pick              --dst-prefix=              --indent-heuristic         --no-max-parents           --quiet                    --until=
--children                 --exit-code                --inter-hunk-context=      --no-merges                --raw                      --walk-reflogs 
--color                    --expand-tabs              --invert-grep              --no-min-parents           --relative-date            --word-diff 
--color-moved              --expand-tabs=             --left-right               --no-notes                 --remotes                  --word-diff-regex=
--color-moved=             --ext-diff                 --max-age=                 --no-patch                 --reverse                  
--color-moved-ws=          --find-copies-harder       --max-count=               --no-prefix                --root                     
--color-words              --first-parent             --max-parents=             --no-renames               --shortstat                
--committer=               --follow                   --merges                   --not                      --show-signature           
--cumulative               --format=                  --min-age=                 --notes                    --simplify-by-decoration
```


