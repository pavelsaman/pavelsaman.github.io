---
layout: page
title:  "wlp2s0, eno1 - where do they come from?"
date:   2019-09-22 09:07:19 +0200
categories: linux
---

I have recently wondered about why network interfaces in Linux have these, at first glance, obscure names. I vaguely remember it used to be more like eth0, wlan0. So I found a bit more info about it, let's summarize it here.

First of all, it really is true that the names used to be simple like eth0, wlan0. These names used to be assigned by the kernel. The problem, however, was that if more interfaces were present (say more enthernet interfaces), the names could change from boot to boot. And that is not very useful since other settings like firewall rules could rely on stable naming schemes.

Alright, so there's obviously a need for stable names that do not change from boot to boot. Therefore there's udev that is nowadays used for assigning persistent names to network interfaces. There's a [good document](https://www.freedesktop.org/wiki/Software/systemd/PredictableNetworkInterfaceNames/) about it.

I think the key part is the algorithm used for determining the names:

```
1. Names incorporating Firmware/BIOS provided index numbers for on-board devices (example: eno1)
2. Names incorporating Firmware/BIOS provided PCI Express hotplug slot index numbers (example: ens1)
3. Names incorporating physical/geographical location of the connector of the hardware (example: enp2s0)
4. Names incorporating the interfaces's MAC address (example: enx78e7d1ea46da)
5. Classic, unpredictable kernel-native ethX naming (example: eth0)
```

It would also be useful to see this in more detail, let's have a look into the [code](https://github.com/systemd/systemd/blob/master/src/udev/net/link-config.c#L403). There I can see the case statement that goes like this:

```
1. ID_NET_NAME_FROM_DATABASE
2. ID_NET_NAME_ONBOARD
3. ID_NET_NAME_SLOT
4. ID_NET_NAME_PATH
5. ID_NET_NAME_MAC
```

Knowing this, I'll find out what's the situation on my system. I'll show the process for my wifi interface.

First, I get the name of such an interface:
```bash
$ ip a
...
3: wlp2s0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
...
```

Then, I will find out more about my interface using utility `udevadm`:

```bash
$ udevadm info --path=/sys/class/net/wlp2s0
P: /devices/pci0000:00/0000:00:1c.3/0000:02:00.0/net/wlp2s0
L: 0
E: DEVPATH=/devices/pci0000:00/0000:00:1c.3/0000:02:00.0/net/wlp2s0
E: DEVTYPE=wlan
E: INTERFACE=wlp2s0
E: IFINDEX=3
E: SUBSYSTEM=net
E: USEC_INITIALIZED=4181766
E: ID_NET_NAMING_SCHEME=v240
E: ID_NET_NAME_MAC=wlx94659c7f5855
E: ID_OUI_FROM_DATABASE=Intel Corporate
E: ID_NET_NAME_PATH=wlp2s0
E: ID_BUS=pci
E: ID_VENDOR_ID=0x8086
E: ID_MODEL_ID=0x095a
E: ID_PCI_CLASS_FROM_DATABASE=Network controller
E: ID_PCI_SUBCLASS_FROM_DATABASE=Network controller
E: ID_VENDOR_FROM_DATABASE=Intel Corporation
E: ID_MODEL_FROM_DATABASE=Wireless 7265 (Dual Band Wireless-AC 7265)
E: ID_MM_CANDIDATE=1
E: ID_PATH=pci-0000:02:00.0
E: ID_PATH_TAG=pci-0000_02_00_0
E: ID_NET_DRIVER=iwlwifi
E: ID_NET_LINK_FILE=/usr/lib/systemd/network/99-default.link
E: ID_NET_NAME=wlp2s0
E: SYSTEMD_ALIAS=/sys/subsystem/net/devices/wlp2s0 /sys/subsystem/net/devices/wlp2s0
E: TAGS=:systemd:
```

I can already see that the first match is `ID_NET_NAME_PATH`, therefore such a name is used. If I look into `lspci`:

```bash
$ lspci
...
02:00.0 Network controller: Intel Corporation Wireless 7265 (rev 59)
...
```
I can see the interface is on bus 2, slot 0, so then I just need to figure out how udev worked from that to `wlp2s0`.

It turns out the naming scheme works like this:
1. if it's ethernet, the prefix will be `en`
2. if it's wlan, the prefix will be `wl`
3. if it's wwlan, the prefix will be `ww`
4. there could also be `sl` for serial line IP (slip), which I don't really know what it means now

Then the other letters will be:
1. `p` for bus
2. `s` for slot
3. `f` for function
4. `x` for MAC address
5. and some other options as well that I'm not familiarized with for now

In my case, it's a wlan interface, therefore we start with `wl` prefix, then the name is determined based on rule number 4 (`ID_NET_NAME_PATH`) from the above algorithm. The bus number is 2, therefore `p2` and the slot is 0, therefore `s0`. It leaves me with the name `wlp2s0`.

### Changing the Name

The names should be persistent just based on the automatic procedure described above, yet for my home use on my laptop, I'd like to have a different name, preferably like the historic names (wlan0, etc.).

Udev also allows for custom naming schemes that take precedence over the algorithm described above. I did exactly that.

It really boils down to only one file in `/etc/udev/rules.d`:

```bash
$ cat 20-wlan.rules 
SUBSYSTEM=="net", DEVPATH=="/devices/pci0000:00/0000:00:1c.3/0000:02:00.0/net/*", NAME="wlan0"
```

I just used information provided by `udevadm` utility. It's already shown above.

On next boot, my wlan interface will be again named `wlan0`. Another look with `udevadm` shows an alias:

```bash
$ udevadm info --path=/sys/class/net/wlan0
P: /devices/pci0000:00/0000:00:1c.3/0000:02:00.0/net/wlan0
L: 0
E: DEVPATH=/devices/pci0000:00/0000:00:1c.3/0000:02:00.0/net/wlan0
E: DEVTYPE=wlan
E: INTERFACE=wlan0
E: IFINDEX=3
E: SUBSYSTEM=net
E: USEC_INITIALIZED=4285178
E: ID_NET_NAMING_SCHEME=v240
E: ID_NET_NAME_MAC=wlx94659c7f5855
E: ID_OUI_FROM_DATABASE=Intel Corporate
E: ID_NET_NAME_PATH=wlp2s0
E: ID_BUS=pci
E: ID_VENDOR_ID=0x8086
E: ID_MODEL_ID=0x095a
E: ID_PCI_CLASS_FROM_DATABASE=Network controller
E: ID_PCI_SUBCLASS_FROM_DATABASE=Network controller
E: ID_VENDOR_FROM_DATABASE=Intel Corporation
E: ID_MODEL_FROM_DATABASE=Wireless 7265 (Dual Band Wireless-AC 7265)
E: ID_MM_CANDIDATE=1
E: ID_PATH=pci-0000:02:00.0
E: ID_PATH_TAG=pci-0000_02_00_0
E: ID_NET_DRIVER=iwlwifi
E: ID_NET_LINK_FILE=/usr/lib/systemd/network/99-default.link
E: ID_NET_NAME=wlp2s0
E: SYSTEMD_ALIAS=/sys/subsystem/net/devices/wlan0
E: TAGS=:systemd:
```

And my interface is renamed:

```bash
$ ip a
...
3: wlan0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
...
```
