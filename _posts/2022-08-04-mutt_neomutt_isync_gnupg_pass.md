---
layout: post
title: Open Source Development with mutt, neomutt, isync(mbsync), gnupg and pass
subtitle: For GMAIL
---

Lightweight Terminal User Interface(TUI) based email clients are good for open source development.
Mutt is one such thing, and do not suck like other email clients.

In this blog we will install and configure mutt and other related applications.

# Install supporting packages #

```
$ sudo apt install mutt neomutt msmtp isync gnupg pass
```

# Configure GPG Key #

GPG is required to encrypt the password.
Create GPG Key.

```
$ gpg --full-generate-key
gpg (GnuPG) 2.2.27; Copyright (C) 2021 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
  (14) Existing key from card
Your selection? 1
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (3072) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0
Key does not expire at all
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: Example Name
Email address: example.name@gmail.com
Comment: 
You selected this USER-ID:
    "Example Name <example.name@gmail.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the               -----------------
disks) during the prime generation; this gives the random number                   |Select secured |
generator a better chance to gain enough entropy.      <---------------------------|passphrase for |
We need to generate a lot of random bytes. It is a good idea to perform            |the email.     |
some other action (type on the keyboard, move the mouse, utilize the               -----------------
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: key 9F8789C146B94BD0 marked as ultimately trusted
gpg: revocation certificate stored as '/home/example/.gnupg/openpgp-revocs.d/84A68802B62844FB8781B1EF9F8789C146B94BD0.rev'
public and secret key created and signed.

pub   rsa4096 2022-08-06 [SC]
      84A68802B62844FB8781B1EF9F8789C146B94BD0
uid                      Example Name <example.name@gmail.com>
sub   rsa4096 2022-08-06 [E]

```

# Encrypt the gmail password for mbsync #

Before downloading emails to use in offline mode, we have to encrypt the password
using "pass".
```
                                                                --------------------------------------
$ pass init 84A68802B62844FB8781B1EF9F8789C146B94BD0  <-------- | The number is from gpg public key. |
                                                                --------------------------------------
$ pass insert mbsync/gmail
```

# Configure mbsync(isync) #

Configure the mbsync in $HOME/.mbsyncrc. This configuration also includes nested gmail labels.
Download the configuration file <a href="https://github.com/shivamurthyshastri/opensdev.github.io/blob/master/files/mbsync_neomutt/mbsyncrc" target="_blank" ref="noreferrer">mbsyncrc</a> and rename it to ".mbsyncrc".

Download all the emails for offline use in a directory '$HOME/Mail'.
The same directory can be used for mutt/neomutt.
```
$ mkdir $HOME/Mail
$ mbsync --all
```

for downloading specific label.
```
$ mbsync amazon
```

# Encrypt the gmail password for mutt #

Now, we encrypt the gmail password for mutt/neomutt.

```
$ mkdir -p $HOME/.config/neomutt
$ vim $HOME/.config/neomutt/password
```

add these two lines:
```
set smtp_pass="sss!1234"
set imap_pass="sss!1234"
```

Encrypt the password:
```
$ gpg -r example.name@gmail.com -e $HOME/.config/neomutt/password
$ rm -rf rm -rf $HOME/.config/neomutt/password
```

Add the following line in neomut configuration file "$HOME/.config/neomutt/neomuttrc".
```
source "gpg -dq $HOME/.config/neomutt/password.gpg |"
```

# Configure mutt/neomutt #

Create the directory and file.

```
$ mkdir $HOME/.config/neomutt/cache
$ touch $HOME/.config/neomutt/certificates
```

Download and copy <a href="https://github.com/shivamurthyshastri/opensdev.github.io/blob/master/files/mbsync_neomutt/neomuttrc" target="_blank" rel="noreferrer">neomuttrc</a>, <a href="https://github.com/shivamurthyshastri/opensdev.github.io/blob/master/files/mbsync_neomutt/color.mutt" target="_blank" rel="noreferrer">color.mutt</a>, <a href="https://github.com/shivamurthyshastri/opensdev.github.io/blob/master/files/mbsync_neomutt/sidebar.mutt" target="_blank" rel="noreferrer">sidebar.mutt</a>, and <a href="https://github.com/shivamurthyshastri/opensdev.github.io/blob/master/files/mbsync_neomutt/signature.mutt" target="_blank" rel="noreferrer">signature.mutt</a> inside $HOME/.config/neomutt.

Now, neomutt should work.
