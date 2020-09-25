---
title: Kernellib Directory Structure
layout: default
---

Apps using the Kernellib have an unusual and distinctive directory structure. Some of it is convention, but a lot is enforced by the Kernellib itself. Some, of course, reflects DGD and LPC.

## The Repo

A source repository (or just a Zipfile or similar archive) containing your app will contain a few distinctive things. There will usually be a file with the extension .dgd, which is normally your [DGD configuration file](https://ChatTheatre.github.io/lpc-doc/dgd/config_file.html).

Sometimes you'll see the DGD root files (mentioned below) directly at the top level. But a well-structured DGD application will keep that configuration file ***out*** of DGD's own addressable space, just to make sure nothing running inside DGD can change DGD's own settings or permissions. So you'll usually have a DGD root directory. The DGD application runs inside that root directory.

This is a lot like a ["chroot jail"](https://en.wikipedia.org/wiki/Chroot) if you're familiar with them. It's also similar to a trick done by a lot of games (e.g. Blizzard games) and many Java applications. By having an internal only-for-this-app file system, you limit damage done by poor security and avoid having to deal with outside parts of the host machine you don't care about.

The DGD root directory is called "skoot" for SkotOS or "root" for eOS. You can find it by checking the "directory" entry in the DGD config file.

## The DGD Root

Inside a DGD app, paths are given relative to the DGD root. So if you see a path like "/kernel/include/std.h", that means it would be under the DGD root directory, perhaps at "root/kernel/include/std.h".

The Kernellib itself normally lives in /kernel. Users can each have directories in /usr. Often libraries get directories under /usr as well, making them a sort of "virtual user".

Here's the breakdown:

* /kernel - The Kernellib itself
** /kernel/data - Kernellib Light-Weight Objects (simple structured types in DGD)
** /kernel/lib - Kernellib Inherited objects (libraries and parent objects, roughly)
** /kernel/obj - Kernellib Objects (meant to be instantiated)
** /kernel/sys - Kernellib System Objects, a.k.a. Daemons (roughly like singletons and/or manager objects)
* /include - Top-level include files, often giving information about the Kernellib or DGD itself
* /usr - User directories, both for literal users and system libraries that act like virtual users
** /usr/System - The core, most privileged setup code
** /usr/admin - A (normally empty) directory for the first and most privileged user, traditionally called "admin"

### The System User

One doesn't normally log in as a "System" user. Instead, the /usr/System directory and its subdirectories are where the game's first setup is performed, loading various other libraries and data.

If you have a file called /usr/System/initd.c, it acts as a sort of "main()" object for a Kernellib application, being created when the application first starts.

### The 'admin' User

When you first set up a new Kernellib-based application, you should first create a new privileged 'admin' user. The Kernellib hardcodes that 'admin' has extensive privileges to do everything. You should keep the password of this user carefully secret, and eventually you may want to prevent them from being able to log in at all. The "admin" user is very similar to the "root" user on Mac OS and Unix-based computers.
