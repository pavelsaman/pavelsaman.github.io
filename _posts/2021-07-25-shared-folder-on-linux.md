---
layout: page
title:  "Shared Folder on Linux"
date:   2021-07-25 17:15 +0200
categories: linux
---

When I say "shared folder", I imagine a folder where I can share data with other system users, but where there are also some restrictions in place. Let's see how this could be achieved on Linux systems.

What comes to mind first is where to create such a folder. I'll go with `/home/shared` location, but I guess some other places could be used as well (e.g. under `/var`).

What I'd like to have is a folder where I can share data between users; the users should be able to create new files in the folder, view files frcreated by other users, but not be able to delete other user's files.

Let's start from the beginning:

```
# umask
0022
```

The umask is 0022 for the root.

```
$ umask
0022
```

And for users as well.

When I create the new shared folder:

```
$ sudo mkdir /home/shared
```

I know that the permissions on the folder will be `rwxr-xr-x`. That's not exactly what I want, because as a normal user I'd not be able to create anything in that folder. It's not shared at all yet.

I'll change the group ownership to `users` and allow write permissions for the group:

```
$ sudo chown :users /home/shared
$ sudo chmod g+w /home/shared/
```

That's better, but not what I want, because when a user creates a new file inside `/home/shared/`, it will belong to the user and the user's primary group.

To change that, I'll set the group id bit:

```
$ sudo chmod g+s /home/shared
```

That will leave me with `rwxrwsr-x` permissions. When a user creates a new file in `/home/shared/`, that file will belong to the same group that `/home/shared/` belongs to:

```
$ cd /home/shared
$ touch a
$ ls -l
total 0
-rw-r--r-- 1 pavel users 0 Jul 25 17:24 a
```

That solves the problem from above. But it also introduces another problem that users can now delete files created by other users. If I log in as a different user, the deletion of file `a` will succeed:

```
$ su test
$ rm a
$ ls -l
total 0
```

And that is what I don't really want. I want only owners of files (or root of course) to be able to delete what they created. I'll have to set the sticky bit:

```
$ sudo chmod +t /home/shared
```

The resulting permissions will read `rwxrwsr-t`; the last `t` stands for the sticky bit.

When I know return to the previous situation when a user wants to delete a file created by a different user, it won't succeed anymore:

```
$ cd /home/shared
$ touch a
$ su test
$ rm a
rm: cannot remove 'a': Operation not permitted
```

All this also means that all users that should be able to have access to the shared folder are added to `users` group. You can check that with `groups` command for instance:

```
$ $ groups test 
users test
```

Of course, I can also play with other permissions. The setup as it is now means users can't edit each other's files. I want to have it that way, but if that's not desirable, you'd need at least `rw-rw-r--` permissions on files inside `/home/shared/`.
