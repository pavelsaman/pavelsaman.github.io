---
layout: page
title:  "Public IP with curl and bash"
date:   2021-07-24 13:30 +0200
categories: linux
---

Getting a private IP address is just a command away in Linux. But often that private IP address is valid only on a particular network you are connected to; hence your public IP address is different. I want to focus on the latter today.

There are numerous online services you can use to figure out your public IP address. I list some of them here:

- [https://ifconfig.me/](https://ifconfig.me/)
- [https://ipecho.net/plain](https://ipecho.net/plain)
- [https://icanhazip.com](https://icanhazip.com)
- [https://ident.me/](https://ident.me/)

Let's try to use one of them with curl:

```
$ curl -s ifconfig.me
```

That will retrieve a public IP address for me.

More interesting example could be a situation when I also use tor. Of course, I can use the same services listed above, but I also need to take into account that tor runs on a localhost.

In curl, I'd type:

```
$ curl -s --socks5-hostname 127.0.0.1:9100 ifconfig.me
```

That will work if tor is running on localhost on port 9100. If it doesn't work for you, check the tor config in `/etc/tor/torrc`.

This second curl command with tor can return an IP address like `217.79.178.53`. I can check that, for example on [https://ipinfo.io/217.79.178.53](https://ipinfo.io/217.79.178.53), and see it does really belong to tor. There is also a website provided by Tor Project that tells you whether or not you're using tor at the moment. Check that here: [https://check.torproject.org/](https://check.torproject.org/).

To put everything together, I wrote a tiny bash function:

```bash
publicip () {
    ip=$(curl -s ifconfig.me)
    ip_proxy=$(curl -s --socks5-hostname 127.0.0.1:9100 ifconfig.me)
    [[ -n $ip ]] && printf "%s\n" $ip
    [[ -n $ip_proxy ]] && printf "%s\n" $ip_proxy
    return 0
}
```

I put that in `/etc/bash/bash_functions` and source the file from `~/.bashrc`. It makes the function available in my terminal:

```
$ publicip
217.79.178.53
```

Then even a public IP address is just a command away :)
