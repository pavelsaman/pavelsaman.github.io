---
layout: page
title:  "Managing Fonts in Linux"
date:   2020-07-23 18:20:19 +0200
categories: linux
---

I've recently updated my Manjaro distribution and lo and behold some of my fonts got broken. It brought me a few very unpleasant moments. In case anybody else (or myself) needs this in the future, I'm now bringing this short article.

First off, my problem was eventually solved by just installing the following package:

```
$ pacman -Qs ttf-dejavu
local/ttf-dejavu 2.37+18+g9b5d1b2f-2
    Font family based on the Bitstream Vera Fonts with a wider range of characters
```

That was easy, but I had to first find out it was because of this (that was a little bit less easy :)).

Anyway, another way to go about it is download your favourite fonts and use them.

I found this [Google site](https://fonts.google.com/) full of fonts. They are all free and open source, so you can easily use them.

Let's say you like this [Roboto](https://fonts.google.com/specimen/Roboto?preview.text=cat+.bashrc&preview.text_type=custom&query=roboto) fonts.

First step would be to find out whether or not you already have them. Sure, you can go to some GUI app and find it out there, but I guess typing `fc-list` and using grep can save you a few seconds.

Now, let's suppose you don't have the fonts and want to be able to use them. So obviously the first step is to download them from the Google site.

Then it depends what users want to use the fonts. If you want them to be accessible globally, you'll use `/usr/share/fonts` dir:

```
$ cd /usr/share/fonts
# cp ~/Desktop/Roboto.zip .
# unzip Roboto.zip
# rm *.zip
```

That really is all. Now you just need to switch to using them.

However, you also might want the fonts to be available just for you. In this case, you'd use your home directory like so:

```
$ mkdir .fonts
$ cd .fonts/
$ cp ~/Desktop/Roboto.zip .
$ unzip Roboto.zip
$ rm *.zip
```

I first create a new hidden directory in my home directory. You obviously skip this step if you have done this already. The rest is the same like in the first example. After doing so, your user can use Roboto fonts.

I suppose that time spent trying to fix my fonts was not all useless, at least I learnt another little piece in Linux.
