---
layout: page
title:  "Checking Status Codes with curl"
date:   2021-06-27 15:45 +0200
categories: testing linux
---

It might be sometimes useful to check a status code of an endpoint. Sure, there're many clients like Postman, but you have to install them, open them, navigate the GUI. All that usually takes quite some time, especially when you're in a hurry. On the other hand, a command line is (should be :)) available all the time, so why not use this environment to check the status code?

I think the most known command line tool for such purposes is [curl](https://curl.se/), it is so uniquitous that even some other tools like the above-mentioned Postman can export http requests into curl commands. However, I won't start with that because it's better to learn the basics before blindly copying commands from somewhere.

In these example, I'll mostly work with two sites, my blog on https://pavelsaman.github.io/ and https://petklub.cz/ which happens to be a site that will return 403 unless you use my company VPN.

If I simply use curl and some URL:

```
$ curl https://pavelsaman.github.io/
```

it will return the content available on that endpoint. Since my blog site is a html page, I'll get exactly that.

This is still useful, but I'm interested only in status codes now, because that's what might serve as an indicator if the site is up and running.

I can do that with -w option:

```
$ curl -w "%{http_code}" https://pavelsaman.github.io/
```

This gets me both the html document as well as a status code.

Let's redirect that html document to `/dev/null`, that's not something I want to see right now:

```
$ curl -w "%{http_code}" -o /dev/null https://pavelsaman.github.io/
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  5535  100  5535    0     0  78043      0 --:--:-- --:--:-- --:--:-- 79071
200
```

As you can see, I don't get the html document, but I still see some curl statistics about the request and response. That's not something I want to see, let's instruct curl to hide it:

```
$ curl -w "%{http_code}" -o /dev/null -s https://pavelsaman.github.io/
200
```

Great, so I finally get only the status code :)

If I try the same with the petklub site, I should get 403 status code:

```
$ curl -w "%{http_code}" -o /dev/null -s https://petklub.cz/          
403
```

Everything seems just fine at this point.

From now on, it really depends what I want to do, perhaps I can write a little bash script that takes some action based on the result:

```bash
#!/bin/bash

status_code=$(curl -w "%{http_code}" -o /dev/null -s https://petklub.cz/)
if (( status_code != 200 )); then
    echo "health check failed, the site is not accessible"
fi
```

I can tweak it to make it work on different test environments:

```bash
#!/bin/bash

declare -A urls=( ["dev"]="https://petklub.cz" ["prod"]="https://hyperinzerce.cz" )
status_code=$(curl -s -w "%{http_code}" -o /dev/null ${urls[$ENV]})
if (( status_code != 200 )); then
    echo $status_code
    exit 1
fi
```

and run it for e.g. prod environment:

```
$ ENV=prod ./healthcheck
```

which will exit with exit code of 1 if the site did not return 200. That could be useful in that I can for example stop a pipeline from running because now I know the site is not accessible, so it makes no sense to continue working with it. That can save some time.

A little bash script with a curl command can go a long way if it's used appropriately. The same task could of course be accomplished in various programming or scripting languages, but that only makes it more complicated and dependent on a particular environment (e.g. I'd have to install some packages like axios from npm registry when scripting this in JavaScript.)
