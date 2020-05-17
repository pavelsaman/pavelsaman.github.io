---
layout: page
title:  "Local DNS Caching with dnsmasq"
date:   2020-05-17 13:00:19 +0200
categories: linux
---

I've recently started using dnsmasq for local DNS caching, so in order to review the whole setup, I've decided to write this article. It's all much easier than you might think, just a few keystrokes away from using a local DNS cache and even DNS over a secure protocol, which as a whole can improve both performance and security.

[dnsmasq](https://en.wikipedia.org/wiki/Dnsmasq) provides a couple of related features for small networks, I'll focus on the DNS caching, which is what I use this product for.

On Arch-like systems, the installation is fairly straightforward, you can use pacman for that:

```
# pacman -Su dnsmasq
```

When installed, a few configuration files will be added:

```
$ cd /etc
$ find . -regex '.*dnsmasq.*'
./NetworkManager/dnsmasq.d
./NetworkManager/dnsmasq-shared.d
./dnsmasq.conf
./systemd/system/multi-user.target.wants/dnsmasq.service
```

The main configuration file is `/etc/dnsmasq.conf`. It turns out the simplest setting requires just a few lines to be changed (mostly uncommented) in this file. Without doubt, dnsmasq could be used in various setups, but one commonly used setup would be NetworkManager and dnsmasq. Or throwing secure DNS into the pot, NetworkManager, dnsmasq, and stubby for DNS over TLS. In this case, the setting of dnsmasq would tipically look like this:

```
no-resolv
listen-address 127.0.0.1
server 127.0.0.1#5300
```

These lines mean dnsmasq won't read `/etc/resolv.conf`, will listen on localhost, and will ask on 127.0.0.1 on port 5300 for domains that are not in its cache. If you're using IPv6, you will repeat `listen-address` and `server` key-value pairs.

You might ask how unknown domains are resolved on my local server. That's when stubby comes into the picture.

Again, stubby is installed as a service using pacman, a few files are added into `/etc`:

```
$ find . -regex '.*stubby.*'
./stubby
./stubby/stubby.yml
./systemd/system/stubby.service
./systemd/system/multi-user.target.wants/stubby.service
```

(Notice the .yml config format which is not that common.)

A crucial line in `stubby.yml` is where the server should listen to for requests:

```
listen_addresses:
  - 127.0.0.1@5300
```

(Too bad that dnsmasq uses '#' as a separator between an address and port and stubby uses '@')

Now, both dnsmasq and stubby should be correctly configured, but we also need to configure that all DNS request are sent to 127.0.0.1 where dnsmasq is listening. It usually requires a change in `/etc/resolv.conf` that will look something like this:

```
$ cat /etc/resolv.conf 
nameserver 127.0.0.1
```

And NetworkManager need to be told not to resolve DNS queries with some other nameserver, so you can add a new config file in `/etc/NetworkManager/conf.d/`:

```
$ cat /etc/NetworkManager/conf.d/dns.conf 
[main]
dns=none
```

Now, it should all be set up correctly, you only might need to restart those services whose settings have been changed, e.g. to restart dnsmasq:

```
# systemctl restart dnsmasq
```

To confirm everything works, you can use `dig` against the local address:

```
$ dig @127.0.0.1 google.com
```

and you should see an output similar to this:

```
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 62860
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 27bc24596cd59e4a0ccc0f0a5ec10a273ee7984cd87cbca7 (good)
;; QUESTION SECTION:
;google.com.			IN	A

;; ANSWER SECTION:
google.com.		246	IN	A	172.217.168.206

;; Query time: 210 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Sun May 17 11:55:52 CEST 2020
;; MSG SIZE  rcvd: 93

```

If I shut down dnsmasq, I won't be successful anymore:

```
# systemctl stop dnsmasq
$ dig @127.0.0.1 google.com
;; connection timed out; no servers could be reached

```

On the other hand, if I turn off stubby, I still can get answers from dnsmasq if such domains are in its cache:

```
# systemctl stop stubby
$ dig @127.0.0.1 getdnsapi.net
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 56512
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;getdnsapi.net.			IN	A

;; ANSWER SECTION:
getdnsapi.net.		365	IN	A	185.49.141.37

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Sun May 17 12:01:24 CEST 2020
;; MSG SIZE  rcvd: 58

```

dnsmasq by default remembers 150 names, it could be changed to a higher number (if you have enough RAM) via this option in `/etc/dnsmasq.conf`:

```
cache-size 1000
```

dnsmasq can also log queries and what it does with them, to turn it on, uncomment the following line in `/etc/dnsmasq.conf`:

```
log-queries
```

It then looks something like this in a log:

```
May 16 23:24:14 pavel-pc dnsmasq[3792]: query[A] m.stripe.com from 127.0.0.1
May 16 23:24:14 pavel-pc dnsmasq[3792]: cached m.stripe.com is 34.214.74.199
May 16 23:24:14 pavel-pc dnsmasq[3792]: cached m.stripe.com is 54.185.149.116
May 16 23:24:14 pavel-pc dnsmasq[3792]: cached m.stripe.com is 54.148.144.252
May 16 23:24:14 pavel-pc dnsmasq[3792]: cached m.stripe.com is 52.37.201.165
May 16 23:24:14 pavel-pc dnsmasq[3792]: cached m.stripe.com is 35.163.124.254
May 16 23:24:14 pavel-pc dnsmasq[3792]: cached m.stripe.com is 34.216.228.194
May 16 23:24:15 pavel-pc dnsmasq[3792]: query[A] rest.contextly.com from 127.0.0.1
May 16 23:24:15 pavel-pc dnsmasq[3792]: forwarded rest.contextly.com to 127.0.0.1
May 16 23:24:15 pavel-pc dnsmasq[3792]: query[A] disqus.com from 127.0.0.1
May 16 23:24:15 pavel-pc dnsmasq[3792]: forwarded disqus.com to 127.0.0.1
May 16 23:24:15 pavel-pc dnsmasq[3792]: reply disqus.com is 151.101.128.134
```

It tells you what it did with DNS requests. So it could be a good way to debug what's going on, but for the normal daily usage, this logging will likely be turned off since it can unnecessary fill the log.

One interesting bit is that dnsmasq was not by default enabled as a service, so after a reboot, I couldn't get to any website because no domain was resolved. The thing I had to do was to enable dnsmasq service:

```
# systemctl enable dnsmasq
```

Now the whole setup should be persistent even after a reboot.