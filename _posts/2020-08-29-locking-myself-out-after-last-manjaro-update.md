---
layout: page
title:  "Locking Myself Out After Last Manjaro Update"
date:   2020-08-29 21:30 +0200
categories: linux
---

Last Manjaro update was not very smooth for me because I eventually managed to lock myself out with no option to log back in to my sestem. Not that it was caused by the update itself, it was caused (of course) by me not paying enough attention to what was being updated.

Manjaro works with [.pac* files](https://wiki.archlinux.org/index.php/Pacman/Pacnew_and_Pacsave#Managing_.pac*_files) that basically provide new (another) configuration options as some packages get updated. This system works fine but it needs a human to actually think about it every now and then and merge the configuration files the way one needs.

So, what was the problem with in my particular case?

In the past, I used `pam_tally2` and `pam_cracklib` pam modules to set up logging into the system and new passwords for user accounts. But these two pam modules just become deprecated and superseeded by `pam_faillock` and `pam_pwquality`.

But as luck would have it, I didn't get rid of the old deprecated moduled in various `/etc/pam.d/` files such as `system-login`:

```
auth       required   pam_tally2.so     onerr=fail file=/var/log/tallylog deny=4 unlock_time=30 even_deny_root root_unlock_time=40
```

Next time I was trying to log in, I was just left wondering why my passwords to various user account did not work.

One option to resolve the situation was to boot a live system, chroot into my system and repair these pam files manually. Of course I deleted the new .pacnew files, so it only took me more time to figure out how to go about the whole disaster. Finally, I managed to use `pam_faillock`:

```
auth       requisite                   pam_faillock.so      preauth    deny=4 silent even_deny_root unlock_time=1200 root_unlock_time=1800 fail_interval=1200
auth       [default=die]               pam_faillock.so      authfail    deny=4 even_deny_root unlock_time=1200 root_unlock_time=1800 fail_interval=1200
auth       required                    pam_faillock.so      authsucc    deny=4 even_deny_root unlock_time=1200 root_unlock_time=1800 fail_interval=1200
```

and `pam_pwquality`:

```
password	required	pam_pwquality.so difok=3 minlen=14 dcredit=2 ucredit=1 lcredit=1 ocredit=2 maxrepeat=2 maxclassrepeat=3 enforce_for_root retry=3
```

the way I was used to with the previous, now deprecated, modules.

The takeaways are clear:

- read the release notes
- don't delete files unless you're really sure you'll never need them
- look into .pacnew files more carefully
