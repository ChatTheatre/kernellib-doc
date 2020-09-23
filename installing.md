---
title: Installing the Kernellib
layout: default
---

(Note: these instructions are written with Linux or Mac in mind. Windows will need a bit of adapting. Pull requests welcome!)

Let's assume you already have [DGD working fine](https://ChatTheatre.github.io/lpc-doc/dgd/installing.html). You'll also need Git and Telnet installed.

Ordinarily, the Kernellib will exist at two locations inside your application's directory structure. But it will also decide ***what that structure should be***, so I'll describe it as if you're creating a new Kernellib-based application. For an existing application, you can just copy the newest files into those two directories.

(I'm ignoring the documentation directory, which you might want as well. But the documentation directory is normally used by human readers, so you can copy or update it however you like. You don't need me to tell you how it must be done.)

* Below I'm going to refer to your source directory as "\~/src/". You can put it wherever your want. But you'll want to substitute your actual directory location for "\~/src" wherever you see it.
* I'll also assume you've built a recent version of DGD in "\~/src/dgd", so the binary is in "\~/src/dgd/bin/dgd".
* And you're creating a Kernellib-based app. I'm going to call that directory "\~/src/my_app". But again, call it whatever you want, and be sure to type your location, not mine.

***First:*** Do you have the kernellib files already? Go to an appropriate directory to put source code. Then "git clone" it with the following command: `git clone https://github.com/ChatTheatre/kernellib.git`. This will create a 'kernellib' directory containing the latest code for the kernellib.

***Second:*** Make directories under your app for "state" (temporary files) and "root" (the DGD root directory.) So: cd into your app directory and make those subdirectories if they don't already exist.

***Third:*** Copy the Kernellib's include directory into place. That's probably: `cp -r ~/src/kernellib/src/include ~/src/my_app/root/`.

***Fourth:*** Copy the Kernellib's kernel directory into place. That's probably: `cp -r ~/src/kernellib/src/kernel ~/src/my_app/root/`.

***Fifth:*** Create a configuration file. Go into your application directory. Copy the kernellib config file into place and then edit it. `cp ~/src/kernellib/kernel.dgd ./my_app.dgd` and then `vi my_app.dgd` (or your favourite other editor rather than vi.) Once you're editing the file, set the "directory" entry to "root" so that DGD can find the DGD root directory you made.

***Sixth:*** Run DGD with your configuration file. `~/src/dgd/bin/dgd my_app.dgd`. Now DGD should be running happily there. Open another console window and you can log in as an administrator. Log in as "admin" with a password of your choice. Type "pwd" to make sure it shows /usr/admin. Then you can type "quit" to quit.

Here are those steps again, as commands:

```
git clone https://github.com/ChatTheatre/kernellib.git ~/src/kernellib  # Clone kernellib under ~/src
cd ~/src/my_app
mkdir -p state root/usr/admin
cp -r ~/src/kernellib/src/include ~/src/my_app/root/
cp -r ~/src/kernellib/src/kernel ~/src/my_app/root/
cp ~/src/kernellib/kernel.dgd my_app.dgd
vi my_app.dgd  # Now set "directory" to "root"
~/src/dgd/bin/dgd my_app.dgd
telnet localhost 6047  # Log in as 'admin' and set up a new password; type "quit" to disconnect
```

### Install Notes on Raspberry Pi

(From DGD List Archives, by Erwin Harte.)

```
Message-ID: <51C603FB.9010707@is-here.com>
Date: Sat, 22 Jun 2013 15:07:23 -0500
From: Erwin Harte <harte@is-here.com>
To: "All about Dworkin's Game Driver" <dgd@dworkin.nl>
Subject: [DGD] DGD on Raspberry Pi

If you want a $35 personal DGD server in your back pocket.

     http://www.raspberrypi.org/faqs

Mine is a previous generation Model B with 256MB, the newer ones have 512MB for
the same price, as I understand it. The Model B has Ethernet. I installed
Raspbian (Debian derivative specifically for the rPi) on this:

$ ssh pi@moonpi
pi@moonpi's password:
Linux moonpi 3.6.11+ #456 PREEMPT Mon May 20 17:42:15 BST 2013 armv6l
<...>
pi@moonpi ~ $ git clone https://github.com/dworkin/dgd.git
<...>
pi@moonpi ~ $ git clone https://github.com/dworkin/kernellib.git
<...>

The yacc/bison program isn't part of the default install, so we'll want that:

pi@moonpi ~/dgd/src $ sudo apt-get install bison
<...>

Nothing to configure since this is a Linux install:

pi@moonpi ~ $ cd dgd/src
pi@moonpi ~/dgd/src $ time make
<...>
gcc -O -g  -o a.out alloc.o error.o hash.o swap.o str.o array.o object.o sdata.o
data.o path.o editor.o comm.o call_out.o interpret.o config.o ext.o dgd.o `cat
comp/dgd` `cat lex/dgd` \
           `cat ed/dgd` `cat parser/dgd` `cat kfun/dgd` `cat lpc/dgd` \
           `cat host/dgd` -ldl

real    5m37.262s
user    4m30.360s
sys    0m6.840s

Go back to the kernellib directory:

- Edit doc/kernel/kernel.dgd for your actual location.
- mkdir ../tmp

pi@moonpi ~/kernellib $ ~/dgd/bin/driver doc/kernel/kernel.dgd
Jun 22 19:35:06 ** DGD 1.4.19
Jun 22 19:35:06 ** Initializing...
Jun 22 19:35:07 ** Initialization complete.
<...>

The telnet program isn't part of the default Raspbian install:
pi@moonpi ~/kernellib $ sudo apt-get install telnet
<...>

pi@moonpi ~/kernellib $ telnet localhost 6047
Trying ::1...
Connected to localhost.
Escape character is '^]'.

DGD 1.4.19 (telnet)

login: admin
Pick a new password:
Retype new password:
Password changed.
# status
                                           Server:       DGD 1.4.19
------------ Swap device -------------
sectors:        135 /      1024 ( 13%)    Start time:   Jun 22 19:35:06 2013
sector size:   0.5K
swap average:  0.11, 0.02                 Uptime:       00:00:32

--------------- Memory ---------------    ------------ Callouts ------------
static:      128656 /    256344 ( 50%)    short:         1 (100%)
dynamic:      86856 /    261120 ( 33%) +  other:         0 (  0%) +
              215512 /    517464 ( 42%)                   1 /      100 (  1%)

Objects:         23 /       500 (  5%)    Users:         1 /       40 (  3%)

#

--Erwin
____________________________________________
https://mail.dworkin.nl/mailman/listinfo/dgd
```
