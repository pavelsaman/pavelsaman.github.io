---
layout: page
title:  "How Much Faster Is Local DNS Caching With dnsmasq?"
date:   2020-05-18 20:00:19 +0200
categories: linux
---

In the [previous article](https://pavelsaman.github.io/local-dns-caching-with-dnsmasq.html), I played around with dnsmasq as a tool for setting up local DNS caching. I'll follow up on that and measure how much faster this setup actually is.

First of all, I need a little script that will resolve a name to an IP address:

```python
#!/usr/bin/env python

import socket

print(socket.getaddrinfo('wikipedia.org', 80));
```

The next thing I'm gonna do is turn off dnsmasq and let only stubby do the whole work of resolving names to IP addresses:

```
# systemctl stop dnsmasq
```

I also need to change the stubby config, it now needs to listen on a default port:

```
listen_addresses:
  - 127.0.0.1
```

I can confirm I haven't broken anything vital:

```
$ dig @127.0.0.1 wikipedia.org
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12093
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 8fd433c7f581afc2b4038f3b5ec2c6429cb1855dea9c9811 (good)
;; QUESTION SECTION:
;wikipedia.org.         IN  A

;; ANSWER SECTION:
wikipedia.org.      375 IN  A   91.198.174.192

;; Query time: 2710 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Mon May 18 19:30:42 CEST 2020
;; MSG SIZE  rcvd: 99

```

Everything seems just fine, names are still being resolved as expected.

I'll run the python script 10 times and measure how much time it takes, so I can get some basic statistics on that later on:

```
$ for i in {1..10}; do (time python resolv.py) 2>&1 | grep real; done
real    0m2.749s
real    0m0.081s
real    0m0.171s
real    0m0.133s
real    0m0.082s
real    0m0.112s
real    0m0.120s
real    0m0.087s
real    0m0.091s
real    0m0.120s
```

From these values, mean is **0.375** and median is **0.116**.

Having measured it, I'll switch dnsmasq back on and have a look what the times are with this setup.

```
# systemctl start dnsmasq
```

And I'll change the stubby setting back as well:

```
listen_addresses:
  - 127.0.0.1@5300
```

And restart stubby as well:

```
# sudo systemctl restart stubby
```

Since dnsmasq keeps DNS records in memory, there's no record on wikipedia yet, so I can try my Python script again:

```
$ for i in {1..10}; do (time python resolv.py) 2>&1 | grep real; done
real    0m2.823s
real    0m0.033s
real    0m0.033s
real    0m0.033s
real    0m0.035s
real    0m0.033s
real    0m0.034s
real    0m0.034s
real    0m0.032s
real    0m0.036s
```

Well, the first query is pretty much the same, no wikipedia record was stored yet. But the remaining 9 requests are significantly faster. That's because dnsmasq already cached the first response, so on those other occasions it was able to serve the request from a local cache.

Just to check the statistics again: mean is **0.313** and median is **0.036**. That's roughly **3.2 times faster** than in the previous example. Plus the times seem to be much more clustered together (apart from the first query of course), which might be another benefit.

All in all, it's a good result for just a few minutes of messing around with dnsmasq :)