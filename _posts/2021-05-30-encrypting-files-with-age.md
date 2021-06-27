---
layout: page
title:  "Encrypting Files with age"
date:   2021-05-30 11:40 +0200
categories: other
---

I've recently found [`age`](https://github.com/FiloSottile/age) utility for file encryption. I find it really handy, so let's have a look at one use case where it might be used.

First of all, age is also available in repositories used by pacman:

```
$ pacman -Ss age | grep "/age "
community/age 1.0.0rc.1-2
```

so the installation is as easy as installing a package from pacman repositories.

The way I used age is together with ssh keys.

Let's generate a new ssh key first:

```
$ ssh-keygen -t rsa
```

That will ask you a few questions and then create two files, one with a private key and one with a public key that you'll use for file encryption.

To do that with age, type:

```
$ age -R id_rsa_files.pub file-to-encrypt > encrypted-file.age
```

`id_rsa_files.pub` is of course the file with the public ssh key generated by `ssh-keygen` in the previous step.

So if someone wants to send you some sensitive information, you can share your public ssh key with them and they can do exactly this. When you receive the file, you can decrypt it like so:

```
$ age -d -i id_rsa_files -o decrypted-file encrypted-file
```

`id_rsa_files` is of course your private ssh key generated by `ssh-keygen`. It's important not to share this file with anybody, you should treat it as a secret, just like any password you use.

Of course if you use a passphrase with your ssh keys, this last command will also ask you for that passphrase.

`age` could also be used with simply passphrases, then the command for file encryption would be:

```
$ age -p file-to-encrypt > encrypted-file.age
```

There're a few other use cases and options, you can dive into the documentation [here](https://htmlpreview.github.io/?https://github.com/FiloSottile/age/blob/master/doc/age.1.html).