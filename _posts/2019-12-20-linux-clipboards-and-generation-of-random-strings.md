---
layout: page
title:  "Linux Clipboards and Generation of Random Strings"
date:   2019-12-20 18:07:19 +0200
categories: linux
---

From time to time, I'm facing the following problem: I need to generate a random (is it really random?) string of a certain length. This scenario can happen mainly from two reasons: 1) I want to generate a new password, 2) I want to test some text field, placing in it an extremely long string and submitting the form (developers must love me). The first scenario could be resolved by many tools, e.g. by Firefox (since version 69) password generator or by some other plugin you install into your browser. However, I still like a bit more not to clutter my computer and software with additional plugins and what not, so why not to build a small tool using stuff already present in my operating system?

### Clipboards

For many, it'll be a surprise when I say that there's not only one clipboard. Actually, there might be only one in Windows, I'm not really sure how it works there. In Linux, you get more clipboards, usually 3. The way it works is these clipboards are called selections and are stored in memory of the controlling application, so it's usually Xorg. If this is the case, then you get these 3 selections:

- PRIMARY: this works when you click the middle mouse button
- SECONDARY: never used, but [it seems](https://unix.stackexchange.com/a/420391/367054) custom app-specific keyboard shortcuts could be used here
- CLIPBOARD: that's what most of us use, classic `Ctrl + c` and `Ctrl + v` combination

Having said that, let's have a look on how to use these clipboards.

### xclip

As it turns out, you need to have some tool that can access your selections. Since I use Manjaro, I can use `xclip`:

```
XCLIP(1)                                                             General Commands Manual                                                            XCLIP(1)

NAME
       xclip - command line interface to X selections (clipboard)

```

However, this is hardly the only tool that lets you manipulate your selections. You might be using a different Linux distribution or simply have differents needs, so you find your own tool.

### Analysis

There are a couple of things I want my tool to be able to handle:

- generate strings of a specified length, but default to some value if I don't specify any length
- generate strings consisting of only digits, alphabet characters, and special characters; or any combination of these 3 categories
- print generated string to console if chosen
- place generated strings into CLIPBOARD by default, or only into PRIMARY selection if chosen, or into both PRIMARY and CLIPBOARD selections if specified

These points are what I want to build with my new tool.

### Code

You can see the [final code here on github](https://github.com/pavelsaman/BashWorkspace/blob/master/gen.bash). To explain some bits:

```bash
while getopts "l:cdsp13" opt; do
  case $opt in
    l) length=$OPTARG;;
    c) add_chars ;;
    d) add_digits ;;
    s) add_schars ;;
    p) print=true ;;
    1) primary=true ;;
    3) clipboard=true ;;
  esac
done
```

Here I'm going through all options I want to be able to use. `-l` for custom length, `-c` for alphabet characters, `-d` for digits, `-s` for special characters, `-p` for printing into console, `-1` for placing the final string into PRIMARY selection, `-3` for placing the final string into CLIPBOARD selection.

So you might wonder how `add_*` functions look like, let's have a look:

```bash
# adds characters into char_set array
function add_chars {
  char_set+=({a..z})
  char_set+=({A..Z})
}

# adds digits into char_set array
function add_digits {
  char_set+=({0..9})
}

# adds special characters into char_set array
function add_schars {
  char_set+=('+' '-' '?' ':' '@' '&' '#' '!' '*' '=' '(' ')')
}
```

`char_set` is an array into which I add more characters based on specified options. This array will later be used as a basis for function `gen_random` that will return randomly generated strings consisting of character sets I specified with options. The funtion `gen_random` looks like this:

```bash
# generate random passcode from characters in char_set
function gen_random {
  local n
  local pass
  while (( length > ${#pass} )); do
    n=$((RANDOM % ${#char_set[@]}))
    pass="${pass}${char_set[n]}"
  done
  echo "$pass"
}
```

I simply add randomly chosen characters from `char_set` array into a string that's later echoed.

### Usage

Just a few examples of how I can use my new tool.

```bash
$ gen -d -l 10 -p
1091521924
```

The string will also be in CLIPBOARD.

```bash
gen -d -l 10 -p -1
1091521924
```

The string will be only in PRIMARY selection.

```bash
$ gen -c -d -s -l 15 -p
xddX0m3:rHLeh@Q
```

Use characters, digits, and special characters as a basis for the string generation, make the string 15 characters long, print it into console and place it into CLIPBOARD.

And so on...

### Limitations

A number of limitations that are not limitations to me, but better mention them here:

- be careful about generating password with this tool, it might end up on the console, in console history or in selections; all these places could be read by other programs/people, so it might be a huge security risk
- at the time, you have to install `xclip` in order for this tool to work; `xclip` might not be available in your distribution
- you cannot specify minimum number of a certain character category (e.g. special characters) in the final string; you might end up with no special characters in your final string even though you specify `-s` option
- special characters are hard-coded in `add_schars` function

However, this should hardly be a bigger tool, it already supports my needs pretty good as it is, but feel free to use bits of it to build something better.