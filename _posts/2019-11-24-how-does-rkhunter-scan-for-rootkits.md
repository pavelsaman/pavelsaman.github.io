---
layout: page
title:  "How Does rkhunter Scan for Rootkits?"
date:   2019-11-24 10:07:19 +0200
categories: linux security
---

Not long time ago, I used a useful tool `rkhunter` to scan my system for possible rootkits and other vulnerabilities. The first time I came across rkhunter was on [Arch wiki site](https://wiki.archlinux.org/index.php/Rkhunter). However, I had not much idea about how the scan for rootkits actually works. After some time spent reading, I'd like to share a thing or two about how rkhunter actually works. It might shed some light on the whole idea of rootkits as well.

### Install rkhunter

First, let's get rkhunter on your machine. I'm using Manjaro, so it's easy via its pacman package manager:

```bash
$ pacman -Syu rkhunter
```

My current version is:

```bash
$ pacman -Qs hunter
local/rkhunter 1.4.6-1
    Checks machines for the presence of rootkits and other unwanted tools.
```

[Wikipedia](https://en.wikipedia.org/wiki/Rkhunter) says, rkhunter is included in some Linux distributions, so you might even already have it installed.

### Rootkits - How It Works?

I won't share much about how to use rkhunter, just read the Arch wiki, that's enough information for you to be able to use it. However, I will share a bit about how rootkits work. I can have much more knowledge about this, however, as a general overview, this should be enough, so you get an idea what such a software can do.

A rootkit is in its essence just a piece of software that reside on your machine. It can be one or a couple of files somewhere on your filesystem. As it will turn out later, this is the key information, but let's not skip other interesting parts.

As the name suggest, rootkit is here to gain access to your privileged account. It does so via a variaous means, and I guess I'm not anywhere near to knowing all of them. However, one way could be exploiting a vulnerability of one of your daemons by sending packets that allow the attacker to e.g. escalate permissions. One example from history would be [Adm rootkit](http://virus.wikidot.com/adm) that were able to cause a buffer overflow of a DNS server running on TCP port 53. Another way attackers might use is a simple old social engineering or direct access to a targeted machine.

Advice number one on how not to get any rootkits:

- update your daemons regularly
- don't share your credentials
- use a correctly configured firewall to protect you from traffic that you don't want to receive

Once a rootkit is on your machine, it might do various things depending on what the attackers want it to do. I guess everyone can name a few, but to make this article complete, let's come up with a few ideas:

- it can create a backdoor into your system as in the example of [Beastkit](https://ossec-docs.readthedocs.io/en/latest/rootcheck/analysis-beastkit.html#analysis-beastkit) that installed its own daemon that would be started at boot on a certain port (again, configure your firewall)
- it can tamper with some valid binaries on your system; it turns out this is a standard practise of rootkits because the attacker wants them to be hidden (for obvious reasons), so the rootkits might use their own versions of programs such as `ps`, `top`, `find`, `md5sum` etc. that don't give you real results; some of these tampered versions are created with `$ touch -acmr` options, so you have no way of finding a tampered binary by comparing its time
- it can send information found on your system to the attacker
- it can tamper with your files, delete them, add something into them
- it can replicate itself by scanning suitable ports and ip addresses; again, this seems to be a common behaviour of rootkits

Therefore, advice number two on how to protect yourself would be:

- after a fresh installation of your system, run an integrity check and store this information somewhere outside your machine; therefore you can later compare the versions and find out if you have a tampered software like `netstat`, `ps` and other
- monitor your network traffic and ports; well, you should know what's going on on your machine anyway, but provided you don't monitor your traffic with a tool that's been tampered with, it will prove useful in uncovering rootkits
- monitor processes and daemons on your machine; the same piece of advice from the previous point applies here as well

### Rkhunter - How It Works?

It turns out rkhunter is a Bourne-shell script, so let's find out what it contains. First, it's a pretty long script:

```bash
$ sudo wc -l rkhunter
21646 rkhunter
```

So, enjoy reading it all :) But let's have a look at some parts. If you check the end of the script, you get something like this:

```shell
sudo tail -n 80 rkhunter 
fi


######################################################################
#
# Start of program actions and checks
#
######################################################################


#
# We can now start to run the actions the user has requested on
# the command-line. We run the update type commands first before
# doing any full system check.
#


#
# The user wants to update the O/S and the file properties data.
#

test $PROP_UPDATE -eq 1 && do_prop_update


#
# The user wants to update the supplied RKH *.dat files.
#

test $UPDATE -eq 1 && do_update


#
# The user wants to check for the latest program version.
#

test $VERSIONCHECK -eq 1 && do_versioncheck


#
# The user wants to check the local system for anomalies.
#

test $CHECK -eq 1 -o $ENDIS_OPT -eq 1 && do_system_check


#
# If there were errors or warnings, and the
# log file is to be copied, then log it.
#

COPIEDLOG=""

if [ $RET_CODE -gt 0 -o $WARNING_COUNT -gt 0 ]; then
	if [ $COPY_LOG_ON_ERROR -eq 1 -a $NOLOG -eq 0 ]; then
		COPIEDLOG="${RKHLOGFILE}.`date +%Y-%m-%d_%H:%M:%S`"

		display --to LOG --type INFO --nl SUMMARY_LOGFILE_COPIED "${COPIEDLOG}"
	fi
fi

display --to LOG --type INFO --nl RKH_ENDDATE "`date`"


#
# Now actually copy the log file.
#

test -n "${COPIEDLOG}" && cp -p "${RKHLOGFILE}" "${COPIEDLOG}"


#
# If the user asked to see the logfile, then show it.
#

test $CATLOGFILE -eq 1 && cat "${RKHLOGFILE}"


IFS=$ORIGIFS

exit $RET_CODE
```

That's where it actually starts. The actual check is started by giving rkhunter `--check`, `--check-all` or `--checkall` (see line 20096) option on the command line, therefore, the following part is of interest:

```shell
test $CHECK -eq 1 -o $ENDIS_OPT -eq 1 && do_system_check
```

Let's dig into it a bit more. Function `do_system_check` initializes a few different checks, it all starts on line 18661 and it goes like this:

```shell
	#
	# We start with checks of some of the commands (binaries) and 
	# libraries on the system, to make sure that they have not
	# been tampered with.
	#

	do_system_commands_checks


	#
	# Next are the rootkit checks.
	#

	do_rootkit_checks


	#
	# Next are some network port, and interface checks.
	#

	do_network_checks


	#
	# Next are checks of the files for the local host.
	#

	do_local_host_checks


	#
	# Next are checks on specific applications.
	#

	do_app_checks
```

Let's go further and have a look at how `do_rootkit_checks` looks like:

```shell
do_rootkit_checks() {

	#
	# This function carries out a sequence of tests for rootkits.
	# This consists of the default files and directories check,
	# possible rootkit checks, and checks for malware.
	#

	if `check_test rootkits`; then
		display --to LOG --type INFO --screen-nl --nl STARTING_TEST rootkits
		display --to SCREEN+LOG --type PLAIN --color YELLOW CHECK_ROOTKITS
	else
		if [ $VERBOSE_LOGGING -eq 1 ]; then
			display --to LOG --type INFO --nl USER_DISABLED_TEST rootkits
		fi

		return
	fi


	rootkit_file_dir_checks

	check_test known_rkts || check_test all && keypresspause

	additional_rootkit_checks

	malware_checks

	trojan_checks

	os_specific_checks

	keypresspause

	return
}
```

Again, there are various other functions that get executed. However, to see just a bit, let's see how rkhunter scans the filesystem:

```shell
rootkit_file_dir_checks() {

	#
	# This function performs the check for known rootkit
	# files and directories.
	#

	if `check_test known_rkts`; then
		display --to LOG --type INFO --screen-nl --nl STARTING_TEST known_rkts
		display --to SCREEN+LOG --type PLAIN --screen-indent 2 ROOTKIT_FILES_DIRS_START
	else
		if [ $VERBOSE_LOGGING -eq 1 ]; then
			display --to LOG --type INFO --nl USER_DISABLED_TEST known_rkts
		fi

		return
	fi


	# 55808 Trojan - Variant A

	SCAN_ROOTKIT="55808 Trojan - Variant A"
	SCAN_FILES=${W55808A_FILES}
	SCAN_DIRS=${W55808A_DIRS}
	SCAN_KSYMS=${W55808A_KSYMS}
	scanrootkit

	# ADM Worm

	SCAN_ROOTKIT="ADM Worm"

	ROOTKIT_COUNT=`expr ${ROOTKIT_COUNT} + 1`

	display --to LOG --type PLAIN --nl ROOTKIT_FILES_DIRS_NAME_LOG "${SCAN_ROOTKIT}"

	if [ -f "/etc/passwd" ]; then
		RKHTMPVAR=`grep 'w0rm' /etc/passwd`

		if [ -z "${RKHTMPVAR}" ]; then
			display --to LOG --type PLAIN --result NOT_FOUND --log-indent 2 ROOTKIT_FILES_DIRS_STR 'w0rm'
			display --to SCREEN+LOG --type PLAIN --result NOT_FOUND --color GREEN --screen-indent 4 NAME "${SCAN_ROOTKIT}"
		else
			ROOTKIT_FAILED_COUNT=`expr ${ROOTKIT_FAILED_COUNT} + 1`
			ROOTKIT_FAILED_NAMES="${ROOTKIT_FAILED_NAMES}${SCAN_ROOTKIT}, "

			display --to LOG --type PLAIN --result FOUND --log-indent 2 ROOTKIT_FILES_DIRS_STR 'w0rm'

			display --to SCREEN+LOG --type WARNING --result WARNING --color RED --screen-indent 4 NAME "${SCAN_ROOTKIT}"
			display --to LOG --type PLAIN --log-indent 9 ROOTKIT_FILES_DIRS_STR_FOUND 'w0rm' '/etc/passwd'
		fi
	else
		display --to SCREEN+LOG --type WARNING --result WARNING --color RED --screen-indent 4 NAME "${SCAN_ROOTKIT}"
		display --to LOG --type PLAIN --log-indent 9 ROOTKIT_FILES_DIRS_NOFILE '/etc/passwd'
	fi

	# AjaKit Rootkit

	SCAN_ROOTKIT="AjaKit Rootkit"
	SCAN_FILES=${AJAKIT_FILES}
	SCAN_DIRS=${AJAKIT_DIRS}
	SCAN_KSYMS=${AJAKIT_KSYMS}
	scanrootkit

	# "Adore" Rootkit

	SCAN_ROOTKIT="Adore Rootkit"
	SCAN_FILES=${AKIT_FILES}
	SCAN_DIRS=${AKIT_DIRS}
	SCAN_KSYMS=${AKIT_KSYMS}
	scanrootkit
```

That's what I mentioned at the beginning, rootkits reside somewhere on your filesystem and rkhunter just looks for them. Therefore, if you take `AjaKit Rootkit` as an example and look for `AJAKIT_FILES` and `AJAKIT_KSYMS` variables, you'll get:

```shell
# AjaKit Rootkit
	AJAKIT_FILES="/dev/tux/.addr
		      /dev/tux/.proc
		      /dev/tux/.file
		      /lib/.libgh-gh/cleaner
		      /lib/.libgh-gh/Patch/patch
		      /lib/.libgh-gh/sb0k"
	AJAKIT_DIRS="/dev/tux
		     /lib/.libgh-gh"
	AJAKIT_KSYMS=
```

In function `scanrootkit`, this list of files and locations is looped over and if such a file is found and not whitelisted, it will report not ok status on the output, e.g. it can look like this in the log file:

```
[12:11:05] Checking for Apache Worm...
[12:11:05]   Checking for file '/bin/.log'                   [ Found ]
[12:11:06] Warning: Apache Worm                              [ Warning ]
[12:11:06]          File '/bin/.log' found
```
(In this case I just ran `$ sudo touch /bin/.log` prior to the test)

This happens when the old way scan is in place, not the thorough check, but by default the option `SCANROOTKITMODE=THOROUGH` is commented out in `/etc/rkhunter.conf`.

### Conclusion

I think this article already gets pretty long with all the examples of rkhunter. However, the point here should be that one of the checks rkhunter runs is a simple scan for specific files and folders on your filesystem. Rootkits are analyzed and the location(s) of their files are more or less known, therefore it can be hard-coded into rkhunter and simply checked with `test`.

As for more advice on preventing your machines from rootkits, I don't really have any. I guess, for a personal machine, this is more than enough if you behave yourself on the Internet. Make sure you keep integrity checks of your key binaries right after a fresh installation, don't run any daemons you don't need, use correctly configured firewall, don't share your credentials, monitor your ports, traffic, and filesystem. Doing just some of it regularly should keep you safe and sound :) If you need more thorough checks, use tools such as rkhunter.
