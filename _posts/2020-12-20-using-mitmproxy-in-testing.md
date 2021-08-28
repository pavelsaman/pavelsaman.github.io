---
layout: page
title:  "Using Mitmproxy in Testing"
date:   2020-12-20 11:30 +0200
categories: testing
---

[Mitmproxy](https://mitmproxy.org/) is an open source HTTPS proxy. Something like this could be extremely useful when doing technical testing of web applications. Let's have a look at how to make everything work so you can leverage the power of proxies in your testing as well.

First off, why use proxies in testing? Simply speaking, it's another source of informating for you as a Tester. Sure, you can always use DevTools where you see what's going on on the network, but it doesn't always give you the power you need - filtering, changing requests, etc.

Alright, so when you understand why proxies might be important for you as a Web-app Tester, let's see some options you have.

Probably the easies way is to install an add-on to Firefox, something like [FoxyProxy](https://addons.mozilla.org/en-US/firefox/addon/foxyproxy-standard/), start a proxy locally (burp suite, mitmproxy, any other) and use the add-on to direct all traffic to the local proxy. That's what I use as well, there's nothing wrong with it.

But let's say you want to use the proxy for other HTTP traffic. Perhaps you want to send a request to some web service, e.g. to get current weather in Tallin:

```
$ curl http://wttr.in/Tallinn?0
```

There's no add-on that will direct the traffic to your proxy. Sure, you can issue the following command and it will go first to your proxy:

```
$ curl --proxy http:127.0.0.1:8080 http://wttr.in/Tallinn?0
```

But that's much more typing and I have to remember there's such an option, which I usually don't remember, so it means opening a man page and searching for such an option, which takes time.

That's where mitmproxy and its transparent mode comes in. I guess you can find other such proxies, but I've recently used this one, so I'll focus on this particular proxy.

I strongly recommend reading mitmproxy tutorials on their website, it guides you through the setting, you can pretty much follow it and it will work. So I won't rewrite every single step here. Instead, I'll just show those key steps I had to do in order to make this all work together.

In the following lines, I'll use a linux machine, precisely my beloved Manjaro 20.2, but it pretty much applies to any linux machine using iptables as a firewall. If you use something else, research how to do it with whatever you use. In case you use a Windows machine, I have no idea how something like this is even possible :D I'd try to set up mitmproxy and see what happens, but I suppose you'd need to reside with a proxy add-on (or browser setting in general) only.

### Adding a new user

This is a key step if the mitmproxy is going to be running on the same machine as where the traffic originates. In such a case, you want to redirect all traffic except that traffic coming from mitmproxy process itself. Otherwise there'd be a loop and nothing useful would ever come out of it. A reasonable workaround is to create a new user that will run the mitmproxy and use `owner` module with iptables. You can in theory use root for this, but I like using unprivileged users as much as I can, so I'll go with a new such user.

This is what I did:

```
# useradd -g users --create-home mitmproxy
# usermod --shell /bin/nologin mitmproxy
```

### Iptables rules

Now I want to redirect all http(s) traffic to mitmproxy. I also want to exclude traffic coming from mitmproxy itself. This is a solution:

```
# iptables -t nat -A OUTPUT -p tcp -m owner ! --uid-owner mitmproxy --dport 80 -j REDIRECT --to-port 8888
# iptables -t nat -A OUTPUT -p tcp -m owner ! --uid-owner mitmproxy --dport 443 -j REDIRECT --to-port 8888
```

Notice that I use a non-default port for mitmproxy. The default port for mitmproxy on Linux is 8080, but I have a feeling this is a default port for more services I might run locally, so I changed it permanently in `~/.mitmproxy/config.yaml`.

### Running mitmproxy

Time to run mitmproxy under the new user:

```
$ sudo -u mitmproxy bash -c 'mitmproxy --mode transparent --showhost'
```

I can confirm it runs under this user:

```
$ ps -U mitmproxy
    PID TTY          TIME CMD
  10436 pts/1    00:00:00 mitmproxy
```

### Additional iptables rules

This would normally be more or less enough (again, read the mitmproxy tutorial, there could be more setting, but it didn't apply to my machine), but in case you have some strict iptables rules, you still might get no traffic coming into the mitmproxy.

In such a case, I recommend looking at your iptables rules:

```
# iptables -L -v --line-numbers
```

and perhaps deleting something that might block it. A good candidate is a last rule in the OUTPUT chain that says drop everything:

```
-A OUTPUT -j DROP
```

Try getting rid of it for a while with `-D` option.

Another candidate is a default policy for the OUTPUT chain:

```
# iptables -L -v
...
Chain OUTPUT (policy DROP 2 packets, 120 bytes)
...
```

which will again drop everything that is not accepted by one of the rules in the OUTPUT chain. And if there's no rule that allows the traffic to the localhost, then it won't work again.

Awyway, make sure, there's something like this rule before you drop everything else:

```
-A OUTPUT -d 127.0.0.1 -p tcp --dport 8888 -j ACCEPT
```

### Traffic in mitmproxy

It should now be all, so if I run the above-mentioned curl command, I should see traffic coming through the mitmproxy running locally:

```
$ curl "http://wttr.in/Tallinn?0"
Weather report: Tallinn

                Mist
   _ - _ - _ -  -2..+3 °C      
    _ - _ - _   ↑ 19 km/h      
   _ - _ - _ -  4 km           
                0.0 mm
```

and the traffic in the mitmproxy:

![image](/images/mitmproxy.png)

There's no need to use any add-on anymore, all the traffic is redirected using iptables. I can even prepare a short script that allows me to set it up and tear it down in one command:

```bash
#!/usr/bin/env bash

# limit PATH
PATH="/bin:/sbin:/usr/bin:/usr/sbin"

# set variables
EXCEPT_USER="mitmproxy"
PROXY_PORT=8888

# set iptables rules for proxy
function iptables_add_nat {
    user="$1"
    port="$2"

    # default values
    [[ -z "$user" ]] && user="$EXCEPT_USER"
    [[ -z "$port" ]] && port="$PROXY_PORT"

    # set the rules
    iptables -t nat -A OUTPUT -p tcp -m owner ! --uid-owner "$user" --dport 80 -j REDIRECT --to-port "$port"
    iptables -t nat -A OUTPUT -p tcp -m owner ! --uid-owner "$user" --dport 443 -j REDIRECT --to-port "$port"
}

# flush nat table OUTPUT chain
function iptables_flush_nat {
    iptables -t nat -F OUTPUT   
}

function print_help {
    echo "proxy_rules add|flush [user] [port]"
}

###############################################################################

case "$1" in
    add)
        echo "Adding proxy rules..."
        iptables_add_nat "$2" "$3" && echo "Done."
        ;;
    flush)
        echo "Flushing nat table OUTPUT rules..."
        iptables_flush_nat && echo "Done."
        ;;
    help)
        print_help
        ;;
    *)
        print_help
        ;;
esac

exit 0
```

If I name this script as `proxy_rules`, then I can just run:

```
# proxy_rules add
```

and the iptables rules are all set up. To flush them, and therefore stop the redirection to mitmproxy, I just run:

```
# proxy_rules flush
```

Be aware that this will flush all rules in the nat table in the OUTPUT chain, therefore use my script with care, it might destroy some of your other rules.

All this playing-around with proxies is also a part of testing. It's something Testers focused on web applications should use in their testing as well because it gives them more power, not only in logging what's going on on the network, but also with changing request going to the server, and therefore e.g. avoiding JavaScript and HTML validations on the frontend.
