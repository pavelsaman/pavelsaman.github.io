---
layout: page
title:  "Run Cron Job Every 5 Seconds"
date:   2021-08-04 12:15 +0200
categories: linux
---

Cron is a great tool for running repetitive tasks. However, the problem might be its granularity, the lowest being 1 minute. If you, say, need to run a task every 5 seconds, there's no easy solution in cron. How to solve it then?

Let's see what the problem is exactly.

If I define a cron job like so:

```
* * * * * my-task.sh
```

it will be run every minute. But there's no way to run the task every n seconds. Of course, it also depends on a cron implementation, perhaps some cron implementations allows to operate with seconds as well, but that's not common.

How to solve it then?

There're more solutions, some of them suitable for only some situations. Let's see two possible options.

What comes to mind first is to use other tools and commands. I can use `sleep` command and a simple shell loop:

```
* * * * * for i in {1..11}; do my-task.sh; sleep 5; done
```

Cron will run this task every minute, but in the script I also count to 11 and sleep after every loop, which will take 55 seconds. That put together makes it run every 5 seconds.

However, there's a big but! If `my-task.sh` script takes more than 5 seconds, those 5 second intervals will not hold anymore. Therefore, make sure that `my-task.sh` runs very quickly and can finish in less than 5 seconds in order to stick to those 5 second intervals.

Another, perhaps more clean, solution would be to use systemd timer units. Of course, this solution requires you be on a systemd init system, but that's quite common nowadays, so perhaps that's even your case.

Systemd timer units allow for smaller granularity, so setting up 5 seconds should not be a problem.

If I'm not mistaken, I'm not running on systemd these days, it can look like this:

```
[Unit]
Description=Run task every 5 seconds

[Timer]
OnUnitActiveSec=5s
OnBootSec=5s

[Install]
WantedBy=timers.target
```

This config file will be placed in `/etc/systemd/system/my-task.timer`. Of course, I also need a service that will be executed when the time is up:

```
[Unit]
Description=My task

[Service]
User=pavel
ExecStart=my-task.sh
```

Again, this service config file will be placed in `/etc/systemd/system/my-task.service`.

That looks clean, I like this even more that the shell loop and sleep command, but you have to use systemd for that.

There might be other solutions like the one [here](https://stackoverflow.com/a/9619441), but I find that extremely cryptic, which disqualifies it from implementation.

Either way, n second intervals are still doable even when you run a non-systemd init system and a standard cron.
