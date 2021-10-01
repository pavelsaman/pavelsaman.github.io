---
layout: page
title:  "IOS Basic Commands and Tips"
date:   2021-12-01 21:00 +0200
categories: networking
---

When iOS is mentioned, people usually imagine Apple operating system, I also imagine Cisco operating system for network devices. Although I should probably respect case, then the Cisco system is really called IOS, not iOS. However, let's have a look at some useful commands and tips for working with IOS.

When you connect to a Cisco router, you will see a command prompt like so:

```
Router>
```

`Router` indicates the device hostname and `>` indicates what mode you're in, it's a user mode here, where you can't really do much.

So you decide to enter a privileged mode. That's done by typing:

```
Router>enable
Router#
```

You can see that the prompt has changed to `#`. You don't need to type the whole `enable` word, it's enough to type only letters that uniquely identify a command. In this case typing `en` is enough:

```
Router>en
Router#
```

Then you probably want to enter a global configuration mode where you can change settings that have an effect on the whole device:

```
Router#config terminal
```

or a shorter version will do:

```
Router#conf t
Router(config)#
```

You can see that the prompt has changed again, this time to `(config)#`. Such a prompt indicates the global configuration mode.

The last mode you'll sooner or later need is an interface configuration mode:

```
Router(config)#interface gigabitEthernet 0/0
Router(config-if)#
```

Here I'm saying I want to configure gigabitEthernet 0/0 interface. I can shorten the whole command to:

```
Router(config)#int g0/0
Router(config-if)#
```

`g` is enough to refer to gigabitEthernet, `f` would be fastEthernet, etc.

Then if I want to leave a certain mode and go to a lower one, I type `exit`:

```
Router(config-if)#exit
Router(config)#
```

And if I want to go to the privileged mode from one of the higher ones, I press Ctrl + Z:

```
Router(config-if)#^Z
Router#
```

Apart from being able to shorten commands, what else can make my time on the command line easier?

IOS also offers help. When in doubt about what commands I can use or what arguments a command has, I can type `?`, for example:

```
Router#sh?
show

```

It printed commands that start with `sh`, which can be only `show` command here.

It works with arguments as well:

```
Router(config)#int fastEthernet ?
  <0-0>  FastEthernet interface number

```

I can see what interfaces I can set.

This `?` can be compared to pressing `TAB` twice in the Linux command line environment.

`TAB` key also works in IOS, it autocompletes a command or argument for you. For example, when I type `int` and press `TAB`, it will finish it to `interface`.

One last useful "shortcut" is typing `write` (or just `wri`) instead of `copy run start`. This is a way to preserve configuration between device reloads. Startup configuration is stored in NVRAM and is loaded when the device is booting. If you don't save your current running configuration, it will be lost on the next boot.

## Shortcuts

Just like on the Linux command line, IOS allows for a number of shortcuts, let's see some of them.

Two of the most useful ones are `ArrowUp` and `ArrowDown` for browsing through command history. If you are used to Linux command line, this is nothing new.

Ctrl + R is different from Linux, though. On Linux, it initiates `(reverse-i-search)`, but on Cisco it starts a newline with the same command:

```
Router#sh // Ctrl + R
Router#sh
```

This is useful in situation when Cisco prints log information right on your current line, so you want to move down and see the whole command with no additional text mixed into it.

Ctrl + U works like in Linux, it deletes the whole line from the cursor to the left.

Ctrl + W also works like in Linux, it deletes the preceeding word.

Ctrl + A and Ctrl + E also work same as in Linux, they move cursor to the beginning and end of the line.

These are probably the most useful shortcuts I know of. There are more, but I don't remember them all :)

## Other tips

Sometimes the configuration doesn't go as expected and you need to start again, in such a situation you can type:

```
Router(config)#default int f0/0
```

to set the fastEthernet 0/0 interface to its default state.

Some commands can be entered only in a certain mode, but IOS also has `do` command that overwrites this limitation, for example:

```
Router(config)#do sh run
```

to see running-config even from the global configuration mode.

Sometimes the outputs can contain a lot of information, but you can filter whan you need with a few commands such as:

```
Router#sh run | begin int
```

To print running config from the first occurence of `int`. Or:

```
Router#sh run | include ip
```

to print only lines that contain `ip`.

`|` is a pipe, you probably know that from Linux.

I've already mentioned that you might get log messages printed right onto your current line. You can always move by pressing Ctrl + R, but it's a bit annoying anyway. It's probably better to set synchronous logging, for example for console:

```
Router(config)#line con 0
Router(config-line)#logging synchronous
```

or I can disable logging messages with command:

```
Router(config)#no logging console
```

You can also see I use `no` here as the first keyword. You can type `no` in front of any command you want to cancel.

## Commands

There are obviously hundreds or more of them, but just to mention a few key ones.

```
Router#sh run
```

It prints current running configuration. The command will print a lot of text, but it will stop when the top of the screen is reached, so to move one page down, you press `Space`, and to move one line down, you press `Enter`.

To see a brief summary of interfaces and their ip addresses:

```
Router#sh ip int brief
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            192.168.4.1     YES TFTP   up                    up      
FastEthernet0/1            192.168.3.1     YES NVRAM  up                    up      
Serial0/1/0                unassigned      YES NVRAM  administratively down down    
Serial0/1/1                unassigned      YES NVRAM  administratively down down
```

To see much more information about interfaces including counters:

```
Router#sh int
```

To clear counters (number of packets received, sent, etc.), you can type:

```
Router#clear counters
```

To see a routing table:

```
Router#sh ip route
```

To see an arp table:

```
Router#sh arp
```

To delete arp cache:

```
Router#clear arp-cache
```

To enable some basic encryption so passwords are not directly in the running config and other places:

```
Router(config)#service password-encryption
```

To set a password for the privileged mode:

```
Router(config)#enable secret abc
```

The password is there just in plain text, which is not the best idea, but if you type:

```
Router(config)#enable secret ?
  0      Specifies an UNENCRYPTED password will follow
  5      Specifies a MD5 HASHED secret will follow
  8      Specifies a PBKDF2 HASHED secret will follow
  9      Specifies a SCRYPT HASHED secret will follow
  LINE   The UNENCRYPTED (cleartext) 'enable' secret
  level  Set exec level password

```

there are some other options how to enter a password more securely.

To clear command history:

```
Router#term history size 0
```

It gets deleted by setting it to zero entries.

To see access lists (ACL):

```
Router#sh access-lists 
Standard IP access list 1
    10 permit 192.168.3.2 (463 matches)
Standard IP access list 2
    10 permit 192.168.4.2 (731 matches)
```

To see a content of NVRAM:

```
Router#dir nvram:
Directory of nvram:/

  189  -rw-        1573                    <no date>  startup-config
  190  ----           5                    <no date>  private-config
  191  -rw-        1573                    <no date>  underlying-config
    1  -rw-           0                    <no date>  ifIndex-table
    2  ----          24                    <no date>  persistent-data
    3  -rw-        2945                    <no date>  cwmp_inventory

196600 bytes total (189850 bytes free)
```

To see a content of a flash memory:

```
Router#dir flash:
Directory of flash:/

    1  -rw-    47453032  Jun 15 2015 07:32:46 +00:00  c1841-adventerprisek9-mz.151-4.M8.bin
    2  -rw-        1289  Sep 29 2021 10:53:40 +00:00  startup-config
    3  -rw-        1574  Sep 30 2021 20:52:36 +00:00  laptops-network
    4  -rw-        1627  Sep 29 2021 11:03:28 +00:00  laptops-network-dhcp

64004096 bytes total (16531456 bytes free)
```

I can back up my running config to a flash drive by typing:

```
Router#copy run my-file
Destination filename [my-file]? 
1548 bytes copied in 1.248 secs (1240 bytes/sec)

```

And then when I don't need the backup anymore, I can delete it:

```
Router#del flash:my-file
Delete filename [my-file]? 
Delete flash:/my-file? [confirm]
```

And last but not least to rename my device to something more memorable:

```
Router(config)#hostname Central
Central(config)#
```

There're much more to learn about, but this should be just a brief article about the most basic stuff to be able to get around this command-line operating system. I hope I'll soon share more, perhaps some of the configuration I experiment with in my home lab.
