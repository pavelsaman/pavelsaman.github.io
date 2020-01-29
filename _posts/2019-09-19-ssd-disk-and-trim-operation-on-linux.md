---
layout: page
title:  "SSD and Trim Operation on Linux"
date:   2019-09-19 16:07:19 +0200
categories: linux
---
I've recently stumbled upon the following topic that seems interesting enough, so after some playing around with it, I've decided to summarize my own thoughts and write this article.

### Hardware
Many modern laptops and other computers don't have a classis hard drive with mechanical parts anymore. The reason is simple, we already have newer technology that enables us to write and retrieve data faster. Yes, I'm talking about SSD, solid state drives. How exactly does it work? Are there some limitations? Can a SSD become worse over time? If so, how can we prolong the period when it works fast?

The high level model of a SSD could be as follows: ![image](/images/block.png)

The smallest unit (not shown in the picture) is a single cell, multiple cells form one page and when we put a bunch of pages together, we get the whole grid as in the picture, which is one block. Of course, there are many blocks inside one SSD. These are the innards of a SSD.

However, one of the problems or limitations of SSDs is, while you can read data on the page level, you can erase data only on the block level. The reason for that is the erase operation requires higher voltage that stresses individual cells around the cells that are being erased. This is already too low-level, so I won't follow this line of thinking anymore.

Having said that, when a disk is new with enough empty blocks, write operations are fast. However, consider the following situation. Some data from a block are deleted: ![image](/images/block_1.png)

Pages A to D do not contain any valid data anymore. We also can't just write new data in there, because the pages have to be erased first and for that we need the whole block to be empty (remember we cannot erase single pages, but only whole blocks). So we are in a situation when some already deleted data are blocking some space on the SSD.

What has to happen next is we need to read all the valid (not stale) data from the block and copy it into a new block, erase this one and only then we can write into this block. So it's not only one write operation anymore.

Fortunatelly, there is a background process called **garbage collection**. This process does what I just described and by doing so, it tries to keep as many blocks completely empty for fast write operations. However, there's still a catch.

The catch is when a file is deleted, it's not really deleted from the disk, it's just not visible for the operating system. This is why there are so many tools that can help you recover deleted files. The data is still on the disk and when the garbage collector on the SSD controller kicks in, it still works even with this data we already deleted on the operating system level. Therefore, it hapilly reads and writes data we already deleted.

That's not very efficient and it's believed to wear the SSD more than it's necessary. Therefore, there's a **TRIM command** that helps to remedy the situation.

The TRIM command propagates the information about deleted data from the operation system to the SSD. Therefore, the SSD does know what data are no longed needed and the garbage collector can leave them behind and erase them. The TRIM command doesn't need to be used, but with it, the whole business of freeing up space is more efficient because not as many data is copied to new blocks.

### Software

Alright, this is a quick introduction into the hardware part, let's now have a look into what options we have in terms of software. For that, I'll use my Manjaro Linux distribution:

```bash
$ uname -r
4.19.69-1-MANJARO
```

```bash
$ cat /etc/lsb-release
DISTRIB_ID=ManjaroLinux
DISTRIB_RELEASE=18.1.0
DISTRIB_CODENAME=Juhraya
DISTRIB_DESCRIPTION="Manjaro Linux"
```
The first file we'll have a look at is `/etc/fstab`:
```
$ cat /etc/fstab
UUID=69CD-94C3                            /boot/efi      vfat    defaults,noatime 0 2
UUID=ba8edea0-5acc-42eb-83b4-a902870f12c4 swap           swap    defaults,noatime,discard 0 2
UUID=900c0943-fd01-481f-9fe6-fae3311a8c72 /              ext4    defaults,noatime 0 1
/dev/mapper/luks-11e54aef-4128-4301-9e9a-230c7dbfe1a9 /home          ext4    defaults,noatime 0 2
tmpfs                                     /tmp           tmpfs   rw,size=3G,noexec,nodev,nosuid,nouser,noatime,auto,async,mode=1700 0 0
```
This is actually not a default installation, but I'll get into that. Two things to notice here are:
1. `discard` option
2. LUKS partition

Since there's a LUKS partition, we also need to have a look into `/etc/crypttab`:
```bash
$ cat /etc/crypttab
luks-11e54aef-4128-4301-9e9a-230c7dbfe1a9 UUID=11e54aef-4128-4301-9e9a-230c7dbfe1a9     - luks,discard,timeout=0,tries=5
```

Here, we can again see the `discard` option.

As it turns out, this `discard` option allows for the TRIM command to be executed and therefore pass the information about deleted data down to the disk.

#### Continuous TRIM and Periodic TRIM

Continuous TRIM works in a way that every time data is deleted, the TRIM command is executed. Periodic TRIM is not executed after every deletion, but is usually run via cron, anacron or systemd service. I'll later show how to set it up via systemd units.

Knowing all this, lets run a few more commands to verify it's true and it works the way I explained:
1. let's create a file
```bash
$ yes | dd iflag=fullblock bs=1M count=1 of=trim.test 
1+0 records in
1+0 records out
1048576 bytes (1.0 MB, 1.0 MiB) copied, 0.00320383 s, 327 MB/s
```
The content of the file is just letter y and newline and it's repeted until 1M in size.
2. get more info about the file, especially the offset, length, and number of bytes
```bash
$ filefrag -s -v trim.test
Filesystem type is: ef53
File size of trim.test is 1048576 (256 blocks of 4096 bytes)
 ext:     logical_offset:        physical_offset: length:   expected: flags:
   0:        0..     255:   44077824..  44078079:    256:             last,eof
trim.test: 1 extent found
```
3. let's get the mount point for the file:
```bash
$ df trim.test
Filesystem                                             Size  Used Avail Use% Mounted on
/dev/mapper/luks-11e54aef-4128-4301-9e9a-230c7dbfe1a9  365G   11G  336G   3% /home
```
4. let's read directly from that address:
```bash
sudo dd bs=4096 skip=44077824 count=256 if=/dev/mapper/luks-11e54aef-4128-4301-9e9a-230c7dbfe1a9 | hexdump -C
00000000  79 0a 79 0a 79 0a 79 0a  79 0a 79 0a 79 0a 79 0a  |y.y.y.y.y.y.y.y.|
*
256+0 records in
256+0 records out
1048576 bytes (1.0 MB, 1.0 MiB) copied, 0.00726108 s, 144 MB/s
00100000
```
We can actually see the data from the file here. There's only one line, but the `*` symbolizes the same data just repeats.
5. I'll delete the file, perform some synchonization and run 4. command again:
```bash
$ rm trim.test 
removed 'trim.test'
$ sync
$ echo 1 > /proc/sys/vm/drop_caches
$ sudo dd bs=4096 skip=44077824 count=256 if=/dev/mapper/luks-11e54aef-4128-4301-9e9a-230c7dbfe1a9 | hexdump -C
00000000  79 0a 79 0a 79 0a 79 0a  79 0a 79 0a 79 0a 79 0a  |y.y.y.y.y.y.y.y.|
*
256+0 records in
256+0 records out
1048576 bytes (1.0 MB, 1.0 MiB) copied, 0.00240065 s, 437 MB/s
00100000
```
**As you can see, I've deleted the file first, but I'm still reading the same data from the device.** How come? That's exactly because I have turned off continuous trimming. How? There's no `discard` option for this filesystem in `/etc/fstab`.

In order for the continuous trim to be enabled, `discard` option has to be put in for those partitions in `/etc/fstab` for which you want to enable the command. If you're working with encrypted partitions, you need to consider `/etc/crypttab` as well and put in the `discard` option there as well.
I'll do the change now, reboot and simulate this scenario again.

When I delete the file and read from the address with `dd` command, I get a different output this time:
```bash
sudo dd bs=4096 skip=44077824 count=256 if=/dev/mapper/luks-11e54aef-4128-4301-9e9a-230c7dbfe1a9 | hexdump -C
00000000  3b bb 20 24 d8 0d e0 86  d1 9b 2a 87 d3 cc 85 46  |;. $......*....F|
00000010  22 bb 7e 2a e9 06 f1 d8  24 7a 44 9f c4 e7 0e cf  |".~*....$zD.....|
00000020  42 fe ca 3a 9a 7e c2 e4  4d b8 b9 ea e3 2f 32 d1  |B..:.~..M..../2.|
00000030  95 5d 80 54 66 9e 0c 33  5e e7 b4 a4 76 dd 68 17  |.].Tf..3^...v.h.|
00000040  5d ab 5c 66 25 2e 46 0d  c0 8e 7a b2 24 1b df 1c  |].\f%.F...z.$...|
00000050  18 10 07 ce b3 1b 2a ef  68 52 d1 07 16 36 67 10  |......*.hR...6g.|
00000060  a4 cb 14 29 0e 1e 17 fb  76 9e 15 88 9e b3 a1 32  |...)....v......2|
00000070  78 2c dd 9e 25 26 99 fd  d6 8b 34 cd cf f3 21 ad  |x,..%&....4...!.|
00000080  7f cd 07 57 39 6d 90 07  87 c2 3f cb aa 80 3f 15  |...W9m....?...?.|
00000090  29 e8 a6 18 09 72 d4 c4  2f 58 04 50 5c 99 52 e0  |)....r../X.P\.R.|
000000a0  42 56 ce 28 d0 37 f4 f7  28 f1 4d 9e a7 78 dc 1f  |BV.(.7..(.M..x..|
000000b0  8a 6f 5f 4f 85 22 23 66  da 04 17 59 c9 37 22 86  |.o_O."#f...Y.7".|
000000c0  50 61 d3 6b 0b 55 3c b0  df fd 52 bc d0 59 27 df  |Pa.k.U<...R..Y'.|
# and much more random data
```

The TRIM command was executed, the SSD already knows about the deleted data.

If you want to find out what filesystems have continuous trimming enabled, run `findmnt -O discard`.

#### Periodic TRIM as Systemd Unit

I have continuous trim disabled, but I still want the TRIM command to tell my SSD what data are useless and should not be copied anywhere else. To do that, I've set up (not really, because it's present by default in Manjaro) two systemd units, `fstrim.timer` and `fstrim.service`.

Again, there'll be some slight variations in my example since I've tweaked the default settings a bit. Anyway, let's have a look at the timer first:

```bash
$ systemctl cat fstrim.timer
# /etc/systemd/system/fstrim.timer
[Unit]
Description=Discard unused blocks once a week
Documentation=man:fstrim

[Timer]
OnCalendar=Wed,Sat
AccuracySec=1h
Persistent=true

[Install]
WantedBy=timers.target
```

By default, the timer is disabled because continuous trim is enabled. In such a situation it doesn't really make sense to have the periodic TRIM run as well.
If enabled (`sudo systemctl enable fstrim.timer`), it runs by default once a week (`OnCalendar=weekly`). My change is I run it on Wednesday and Saturday.

Then there's `fstrim.service` systemd unit without which the timer would do absolutely nothing:

```bash
# /usr/lib/systemd/system/fstrim.service
[Unit]
Description=Discard unused blocks on filesystems from /etc/fstab
Documentation=man:fstrim(8)

[Service]
Type=oneshot
ExecStart=/sbin/fstrim --fstab --verbose --quiet
ProtectSystem=strict
ProtectHome=yes
PrivateDevices=no
PrivateNetwork=yes
PrivateUsers=no
ProtectKernelTunables=yes
ProtectKernelModules=yes
ProtectControlGroups=yes
MemoryDenyWriteExecute=yes
SystemCallFilter=@default @file-system @basic-io @system-service
```

`fstrim` command can take `--dry-run` option, so you can actually run in without eny real effect and just check what it does:
```bash
sudo /sbin/fstrim --fstab --verbose --quiet --dry-run
/home: 0 B (dry run) trimmed on /dev/mapper/luks-11e54aef-4128-4301-9e9a-230c7dbfe1a9
/: 0 B (dry run) trimmed on /dev/sda3
/boot/efi: 0 B (dry run) trimmed on /dev/sda1
```

It seems it's working. It simply looks into `/etc/fstab` and trims all mounted filesystems mentioned in `/etc/fstab` on devices that support the discard operation. A good idea is to have a look what mounted filesystems support it:

```bash
$ lsblk --discard
NAME                                          DISC-ALN DISC-GRAN DISC-MAX DISC-ZERO
sda                                                  0      512B       2G         0
├─sda1                                               0      512B       2G         0
├─sda2                                               0      512B       2G         0
├─sda3                                               0      512B       2G         0
└─sda4                                               0      512B       2G         0
  └─luks-11e54aef-4128-4301-9e9a-230c7dbfe1a9        0      512B       2G         0
```

All I'm interested in is `DISC-GRAN` and `DISC-MAX` columns. If both are with non-zero values, just a filesystem supports trimming. Make sure that if you want to disable continuous trim for LUKS partition, there should be no `discard` option for such a partition in `/etc/fstab`, but there should be `discard` option for the LUKS partition in `/etc/crypttab`.

We have confirmed there's no problem, so let's run the command as a service now:
```bash
$ sudo systemctl start fstrim.service
$ systemctl status fstrim.service
● fstrim.service - Discard unused blocks on filesystems from /etc/fstab
       Loaded: loaded (/usr/lib/systemd/system/fstrim.service; static; vendor preset: disabled)
       Active: inactive (dead) since Wed 2019-09-18 15:29:32 CEST; 9min ago
         Docs: man:fstrim(8)
      Process: 1445 ExecStart=/sbin/fstrim --fstab --verbose --quiet (code=exited, status=0/SUCCESS)
     Main PID: 1445 (code=exited, status=0/SUCCESS)
    
    Sep 18 15:29:31 pavel-pc systemd[1]: Starting Discard unused blocks on filesystems from /etc/fstab...
    Sep 18 15:29:32 pavel-pc fstrim[1445]: /: 0 B (0 bytes) trimmed on /dev/sda3
    Sep 18 15:29:32 pavel-pc fstrim[1445]: /boot/efi: 510.7 MiB (535519232 bytes) trimmed on /dev/sda1
    Sep 18 15:29:32 pavel-pc systemd[1]: fstrim.service: Succeeded.
    Sep 18 15:29:32 pavel-pc systemd[1]: Started Discard unused blocks on filesystems from /etc/fstab.
```

The service has run successfully, but there's one small detail. Can you see it?

When run as a command, `fstrim` reported, it can trim `/home` as well, but there's no `/home` when we run it as a service.

We can even have more into the log:
```bash
$ journalctl -b --no-pager | grep fstrim
Sep 18 15:29:31 pavel-pc sudo[1442]:    pavel : TTY=pts/0 ; PWD=/home/pavel ; USER=root ; COMMAND=/usr/bin/systemctl start fstrim.service
Sep 18 15:29:32 pavel-pc fstrim[1445]: /: 0 B (0 bytes) trimmed on /dev/sda3
Sep 18 15:29:32 pavel-pc fstrim[1445]: /boot/efi: 510.7 MiB (535519232 bytes) trimmed on /dev/sda1
Sep 18 15:29:32 pavel-pc systemd[1]: fstrim.service: Succeeded.
```

Nothing.

Well, the problem really is in the systemd service unit, there's a line that says `ProtectHome=yes`, therefore the `fstrim` command was not run for `/home`.

Having said that, let's change it:
```bash
$ sudo systemctl edit --full fstrim.service
# change it in your text editor and save it
$ systemctl cat fstrim.service
# /etc/systemd/system/fstrim.service
[Unit]
Description=Discard unused blocks on filesystems from /etc/fstab
Documentation=man:fstrim(8)

[Service]
Type=oneshot
ExecStart=/sbin/fstrim --fstab --verbose --quiet
ProtectSystem=strict
ProtectHome=no
PrivateDevices=no
PrivateNetwork=yes
PrivateUsers=no
ProtectKernelTunables=yes
ProtectKernelModules=yes
ProtectControlGroups=yes
MemoryDenyWriteExecute=yes
SystemCallFilter=@default @file-system @basic-io @system-service
```

When run now, it takes `/home` into consideration and performes trimming on it as well:
```bash
Sep 18 15:47:29 pavel-pc sudo[2576]:    pavel : TTY=pts/1 ; PWD=/home/pavel ; USER=root ; COMMAND=/usr/bin/systemctl start fstrim.service
Sep 18 15:47:31 pavel-pc fstrim[2579]: /home: 846.6 MiB (887713792 bytes) trimmed on /dev/mapper/luks-11e54aef-4128-4301-9e9a-230c7dbfe1a9
Sep 18 15:47:31 pavel-pc fstrim[2579]: /: 287.2 MiB (301125632 bytes) trimmed on /dev/sda3
Sep 18 15:47:31 pavel-pc fstrim[2579]: /boot/efi: 510.7 MiB (535519232 bytes) trimmed on /dev/sda1
Sep 18 15:47:31 pavel-pc systemd[1]: fstrim.service: Succeeded.
```

### Conclusion

That wraps up this article. It's not really anything difficult, but it requires a little bit of reading and digging around in some files on the system. Once set up, it can run with no manual input whatsoever.

If you need more info, you can always check man pages or Google. I hope I've covered the most important parts of this topic and that my article could be used as a good reference when facing the same need to set this up. When reading about this topic, I particularly liked Arch wiki pages where this is also described. The rest was just looking into a few forum threads and piecing the bits together.