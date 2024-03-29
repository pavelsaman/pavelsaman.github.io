---
layout: page
title:  "Process Connection Log with Awk"
date:   2019-09-20 14:07:19 +0200
categories: linux
---

Awk is a useful tool for text processing. I've recently wanted to process a log generated by Network Manager. I'll show you just a simple awk script that helped me get the data I wanted. I find it useful to get some practice in awk, so I'll try out some awk oneliners here.

The log is in two file in `/var/log`:

```bash
-rw-r--r--   1 root   root            4.6K Sep 20 15:10 NetworkManager-dispatcher.d.log
-rw-r--r--   1 root   root             11K Sep 15 00:00 NetworkManager-dispatcher.d.log.1
```

The file content is in the following format (I deleted the real UUID of my networks):

```
2019-09-20T20:55:14,334287331+00:00 | DEVICE: wlp2s0 | STATUS: down | UUID: xxx | ID: akula | IP4_GATEWAY:  | IP4_NAMESERVERS: 
2019-09-08T08:36:29,311656818+00:00 | DEVICE: eno1 | STATUS: up | UUID: xxx | ID: Wired connection 1 | IP4_GATEWAY: 192.168.1.1 | IP4_NAMESERVERS: 8.8.8.8
2019-09-08T09:15:36,751701614+00:00 | DEVICE: eno1 | STATUS: down | UUID: xxx | ID: Wired connection 1 | IP4_GATEWAY:  | IP4_NAMESERVERS: 
2019-09-08T10:03:08,171797403+00:00 | DEVICE: eno1 | STATUS: up | UUID: xxx | ID: Wired connection 1 | IP4_GATEWAY: 192.168.1.1 | IP4_NAMESERVERS: 8.8.8.8
2019-09-08T11:37:24,046263858+00:00 | DEVICE: eno1 | STATUS: down | UUID: xxx | ID: Wired connection 1 | IP4_GATEWAY:  | IP4_NAMESERVERS: 
2019-09-08T11:48:09,947160296+00:00 | DEVICE: eno1 | STATUS: up | UUID: xxx | ID: Wired connection 1 | IP4_GATEWAY: 192.168.1.1 | IP4_NAMESERVERS: 8.8.8.8
```

For every `up` or `down` status, there's one line with date and time, device name, status, UUID of the network, ID of the network, IPv4 address, and an array of nameservers.

I'll now run a few awk commands to process the file and get some interesting statistics.

#### I want to know how many times I've connected to a network

```bash
$ awk -F'|' '($3 ~ /up/) { c++ } END {print c}' /var/log/NetworkManager-dispatcher.d.log*
```

No need to write `if`, I can just type `(pattern)` and apply `{action}`. 

#### I want to know what networks I've connected to

```bash
$ awk -F'|' '($3 ~ /up/) {net[$4]++} END{for (n in net) {print substr(n,2)}}' /var/log/NetworkManager-dispatcher.d.log*
```

Awk has some functions I can use. I can even create my own functions in awk.

#### I want to know what networks I've connected to + how many times

```bash
$ awk -F'|' '{ntw=substr($4,index($4,":")+2,length(substr($4,index($4,":")+2))-1)} ($3 ~ /up/) {net[ntw]++} END{for (n in net) {print n,net[n]}}' OFS='|' /var/log/NetworkManager-dispatcher.d.log*
```
`OFS` changes the Output Field Separator. I can also beautify the output by getting rif od some whitespaces, etc. by using `substr`, `index`, and `length` functions.

#### I want to add the network ID to the previous output

```bash
$ awk -F'|' '($3 ~ /up/) {net[substr($5,index($5,":")+2,length(substr($5,index($5,":")+2))-1) "|" substr($4,index($4,":")+2,length(substr($4,index($4,":")+2))-1)]++} END{for (n in net) {print n,net[n]}}' OFS='|' /var/log/NetworkManager-dispatcher.d.log*
```

In awk I can really play around with the key of an associative array.

#### If I want to put the previous oneliner into a script file

```awk
#!/usr/bin/awk -f

BEGIN {
	FS="|";
	OFS="|";
}

{
	if ($3 ~ /up/) {
		net[substr($5,index($5,":")+2,length(substr($5,index($5,":")+2))-1) "|" substr($4,index($4,":")+2,length(substr($4,index($4,":")+2))-1)]++;
	}
}

END {
	for (n in net) {
		print n,net[n];
	}
}
```

Awk can be run as a script just like a bash script. This is definitely more neat when the script grows in length.

#### I want to sort the previous output by the number of connections

Provided I saved the previous script as a file `network_connections.awk`

```bash
$ ./network_connections.awk /var/log/NetworkManager-dispatcher.d.log* | sort -t'|' -n -r -k 3
```

The first line is the most used network on my machine.

#### I want to know what IP addresses I've had

```bash
$ awk -F'|' '($3 ~ /up/) {net[$6]++} END{for (n in net) {print substr(n,2)}}' /var/log/NetworkManager-dispatcher.d.log*
```

Awk has associative arrays, so it's easy to use some data as a key.

#### I want to create a file named after an ID of a connection and save all times I've connected to such a network into the file

```bash
$ awk -F'|' '{file_name=substr($5,index($5,":")+2,length(substr($5,index($5,":")+2))-1)} ($3 ~ /up/) {print $1 > file_name}' /var/log/NetworkManager-dispatcher.d.log*
```

#### I want to print only lines with a connection I hand to awk in a variable `NET`

```bash
$ awk -v NET="akula" -F'|' '{ntw=substr($5,index($5,":")+2,length(substr($5,index($5,":")+2))-1)} ($3 ~ /up/ && ntw == NET) {print}' /var/log/NetworkManager-dispatcher.d.log*
```

#### I want to know how many times I've connected to a network and sort it desc

```bash
$ awk -F'|' '($3 ~ /up/) {x[substr($1, 0, index($1, "T")-1)]++} END{for (n in x) {print n,x[n]}}' /var/log/NetworkManager-dispatcher.d.log* | sort -h -k2 -r
```

Awk can create files for me and I can use redirection in awk.

This is how awk works. Just a few notes at the end:
1. awk uses pattern-action patters; patterns are in `()`, actions in `{}`, when a pattern gets evaluated to true, action is taken
2. awk uses associative arrays, so it's easy to use a piece of data as a key
3. there are some built-in variables in awk:
![image](/images/awk_vars.png)
4. it's possible to write a script in awk and save it as a file, the first line would be `#!/usr/bin/awk`
5. awk can redirect output to a file, it's even possible to use shell from within an awk script
6. awk works with regular expressions that are enclosed in `//`, unlike grep for example
7. to hand a variable from shell to awk, I can use option `-v`, such a variable will not be available in `BEGIN` block though
8. awk can also read an env variable `ENVIRON["MY_VAR"]`, this can be used even in `BEGIN` block

All in all, with a couple of characters, awk lets you do a lot with any text. It's worth it to know how to use it.
If you want to see more examples, you can go into my [AwkWorkspace repository on github](https://github.com/pavelsaman/AwkWorkspace) or [see some of my awk libraries](https://github.com/pavelsaman/Awk-lib).