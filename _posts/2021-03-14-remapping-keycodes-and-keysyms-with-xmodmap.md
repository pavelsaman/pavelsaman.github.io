---
layout: page
title:  "Remapping Keycodes and Keysyms with xmodmap"
date:   2021-03-14 17:30 +0200
categories: linux
---

There're some annoying keys on my keyboard that I never use, moreover if I even touch them, it's always by mistake. It'd be nice to remap them to either something more useful or remap them to nothing.

I'm talking about `CapsLock` and `NumLock`. I never use these two keys, so I'll make them innactive.

Firstly, it doesn't hurt to understand a bit about what happens when a key on a keyboard is pressed. It generates a scancode that's sent to a computer, the kernel maps scancodes to keycodes and eventually every keycode is mapped to a certain character called a keysym which is later received by an application. Genrally, there're two mappings, between scancodes and keycodes and between keycodes and keysyms, both could be changed, I'll change the second one in this post.

Let's see my current mapping between keycodes and keysyms (I'm showing only a tiny bit, not the whole output):

```
$ xmodmap -pke
...
keycode  38 = a A a A asciitilde AE ae AE
keycode  39 = s S s S dstroke section ssharp section
keycode  40 = d D d D Dstroke ETH eth ETH
keycode  41 = f F f F bracketleft ordfeminine dstroke ordfeminine
keycode  42 = g G g G bracketright ENG eng ENG
...
```

This basically means that for keycode 38, character "a" is generated. From the second column on after the "=", it shows what is generated if I also press a modifier key like `Shift`, `mode_switch`, or `AltGraph`, for example 5th and 6th column show what character is generated when I press `AltGraph + a` and `AltGraph + Shift + a`.

Another view on the same problem could be with `showkey -k`. That will give me a similar output:

```
$ showkey -k
press any key (program terminates 10s after last keypress)...
keycode  28 release
akeycode  30 press
keycode  30 release
```

So I can see that after pressing "a" on a keybord, keycode 30 was generated. This doesn't really correspond to the above value of 38, but that's only because I have to subtract 8 from whatever code `xmodmap` gives me to arrive at a kernel keycode `showkey` gives me.

The rest is simple, I just need to map `CapsLock` and `NumLock` keys to nothing. I'll first find these two:

```
$ xmodmap -pke | grep -E '(Caps_Lock|Num_Lock)'
keycode  66 = Caps_Lock NoSymbol Caps_Lock
keycode  77 = Num_Lock NoSymbol Num_Lock
```

I'll back up my current mapping in case something goes terribly wrong:

```
$ xmodmap -pke > .xmodmap-original
```

Now I can create a new mapping file, it'll look like so:

```
$ cat .Xmodmap
keycode 66 =
keycode 77 =
```

Finally I just need to make sure this mapping gets used, so let's have a look into `.xinitrc` file (again, I'm showing just a bit of the whole file):

```
$ cat .xinitrc
...
userresources=$HOME/.Xresources
usermodmap=$HOME/.Xmodmap
sysresources=/etc/X11/xinit/.Xresources
sysmodmap=/etc/X11/xinit/.Xmodmap

SESSION=${1:-xfce}

# merge in defaults and keymaps

if [ -f $sysresources ]; then
    xrdb -merge $sysresources
fi

if [ -f $sysmodmap ]; then
    xmodmap $sysmodmap
fi

if [ -f "$userresources" ]; then
    xrdb -merge "$userresources"
fi

if [ -f "$usermodmap" ]; then
    xmodmap "$usermodmap"
fi
...
```

It seems everything's ready, no need to do enything else, because if `.Xmodmap` exists as a file, it will be used to map new keycodes and keysyms. If you don't have `.xinitrc` file prepared like I do, you need to create it on your own (meaning mostly that one condition).

That's basically it, now if I press `CapsLock`, nothing happens, I do not start typing in capital latters. And I never turn off my numpad with `NumLock` since the key is now also mapped to nothing. It's also worth mentioning that if you do the remapping when e.g. `NumLock` is switched off, you won't be able to turn on the numpad.
