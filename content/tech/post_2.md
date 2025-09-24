---
title: "Signing Git Commits with GPG"
date: 2021-02-27T01:48:27+05:00
draft: false
author: "Muhammad Tehami"
tags: ["GPG", "Git", "Security", "Github", "DevOps", "GitOps"]
categories: ["Security"]
---

## Overview

![Git GPG](/images/posts/post_2/git-gpg.jpeg)

### Git

Git is a crucial part of every developer’s life. We have come across from the cumbersome and hectic usage of bad old days versioning practice to a centralized and easy to collaborate process with git.

As the versioning became easier and collaboration is on our fingertips, there comes the issue of authenticity of the work contributed by individuals.

### The Donkey in Lion’s Skin

We all have heard the The Donkey in Lion’s Skin story. The TLDR is a donkey dressed itself in a lion’s skin. Wherever the donkey went, the other animals and villagers feared him. The villagers and other animals thought that he was a real lion. After some time he became bold. Finally one day some people heard him braying. The people ran after him with sticks, and beat him to death. Hence, the poor donkey paid the price for his foolishness.

But what does the donkey story have to do with git? In git, a single contribution is called a commit which holds all the meta data regarding the work being done and by whom. The user meta data which is name and email address are associated with every commit. The problem with this is that anyone can add any user and email address to commit even if the email doesn’t belong to that user.

### Public-key cryptography

A type of cryptography which uses a pair of keys, a public key and a private key; the two keys have the related in a way that, given the public key, it is computationally infeasible to derive the private key. For connection establishment, public-key cryptography enables different parties to communicate securely without having prior access to a secret key (unlike symmetric key cryptography which only involves one key) that is shared, by using one or more pairs (public key and private key) of cryptographic keys. There are lot of tools which implement public key cryptography. One of them is GPG.

### PGP, OpenPGP, GPG

Back in 1990s Phil Zimmermann created a data encryption and decryption computer program called Pretty Good Privacy (PGP) which is currently owned by software security company Symantec. Soon PGP become the de-facto standard in e-mail communication. PGP is a commercial product, and as the opensource software boomed the need for the opensource alternatives to the PGP became inevitable. This gave birth to OpenPGP which is an open source standard that allows PGP to be used in software that is typically free to the public. The term “Open PGP” is often applied to tools, features, or solutions that support open-source PGP encryption technology. GPG is an implementation of the Open PGP standard. The GPG, or GnuPG, stands for GNU Privacy Guard.

![Cut the crap meme](/images/posts/post_2/cut-the-crap.gif)

## Back To The Terminal

### Install GPG

Install GPG depending on the OS you are using: https://gnupg.org/download/

After the installation, verify that you have it working:

```
gpg --version
```

### Creating GPG keypair

```
gpg --full-generate-key
```

Now an interactive console will ask some information.

Use option one (1) RSA and RSA (default), give key size (ideally 4096), set expiry date for example 1y, give your real name and email address and comment (optional).

After this GPG will perform some math behind the key generation which uses some big random prime numbers. It is a good idea to perform some other action (type on the keyboard, move the mouse, utilize the disks) during the prime generation; this gives the random number generator a better chance to gain enough entropy.

I use dd(copy files) and rngd(a random number generator) on linux. You can use any program which does a lot of cpu processing and uses hard drive (Chrome included :D). Use following if you are on server which can’t get enough entropy.

```
sudo yum install rng-tools
sudo systemctl start rngd
sudo dd if=/dev/sda of=/dev/zero
```

Now we have our gpg keypair ready to use. Verify that the key is created successfully:

```
gpg --list-keys
```

### Sign git commit

Make sure your name and email in git configuration is same as you have in the gpg key:

```
git config --global user.name "Your Name"
git config --global user.email "Your Email"
```

Create a directory and initialize git:

```
mkdir repo
cd repo
git init
```

Stage a file:

```
touch file
git add file
```

Now create a signed commit:

```
git commit -S -m "My signed commit" 
```

Check the signed commit:

```
git log --show-signature
```

You successfully signed commit with your private GPG key.

### Distributing public GPG key

It’s time to distribute your public key so that others will know that the commit was actually signed by your private key.

Get your public key:

```
gpg --export --armor "Your Email"
```

Copy the output and go to:

Github -> settings -> SSH and GPG keys.

Click on New GPG Keys button and paste you public key. Click Add GPG key button.

After doing above when you push the code to Github it will show a verified badge against your commit, which shows that this key was actually signed by the corresponding private key of the given public key in Github.

## Caveat

When you don’t provide the -S flag in git commit it’ll not sign the commit. If you want to sign every commit without providing the -S flag explicitly, then modify your global git config file as follows:

Open git config file from ~/.gitconfig and add following lines in it.

```
[commit]
        gpgsign = true
[tag]
        gpgsign = true
```

You can now use usual git commit to create signed commits.

```
git commit -m "Signed commit without -S"
```

![Cartoon meme](/images/posts/post_2/not-that-hard.gif)

### Now what?

Remember that donkey in lion’s skin. Well we don’t have to worry now :D

Thanks for following along, take a look at GPG cheat sheet I’ve created:

https://github.com/t3hami/GPG-cheat-sheet.git

