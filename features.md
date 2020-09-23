---
title: Features of Kernellib
layout: default
---

## Permissions

DGD doesn't have any native idea of permissions within a DGD process. That should sound normal - the same is true of nearly any programming language.

DGD allows an object to query what code has called the current function. Through careful use of this facility, objects can expose public (unprotected) and private or semi-private methods which can only be called by appropriate, known callers.

In general, the Kernellib divides code into "usr" directories, such as the highly-privileged /usr/System directory. Other directories can limit permissions of who can compile and instantiate their objects, and who can call into their API functions.

## Persistence (Which Needs Upgradeability)

One of the major functions of Kernellib is to manage persistence. DGD potentially allows fully recompiling all objects, but in practice that has some limitations. For somewhat-complicated reasons, you can't safely recompile a ***parent*** object that has other objects inherit from it while persisting its state (such as any data fields in the object.)

Persistence means your server can keep running for years without having to reboot, and/or by reloading snapshots of the previous execution state. However, persistence on that time scale requires being able to upgrade your code over time, and upgrading your objects in a running copy of DGD.

The Kernellib handles this by requiring all ***parent classes*** to be ***abstract parent classes***. If it's possible to inherit from an object then you can't instantiate it.

It does this through a combination of naming and object management.

## Naming

DGD normally names an object by where it occurs in the directory hierarchy. An object in /usr/System/initd.c is normally referred to as "/usr/System/initd" by DGD when you deal with it. It's common to define a constant (that is, an ALL_CAPS_NAME) for each object, but the value of that constant is the path to the object's code.

In Kernellib, that name determines some important things about the object's capabilities. For instance, if the name contains /usr/SomeUser, then SomeUser is considered the creator of that object, which determines what files the object can read and write. Different capabilities can be set in Kernellib for different directories, allowing objects from those directories to do (or not do) particular things.

Naming is also used to determine whether a particular object is a parent object (if its path contains /lib/ anywhere), a normal instantiable object (normally uses /obj/ or /sys/ rather than lib) or a lightweight object that doesn't manage its own memory (/data/).

Thus, in Kernellib the naming determines both the ***type*** of the object (parent, child, LWO) and the ***permissions*** for the object (creator, directory.)

## Object Management

DGD has the ability for your application to register an Object Manager. In a language where you can fully recompile any object, it's useful to tell when this has happened and update appropriately. The Object Manager receives these events from DGD and may respond to them.

The Kernellib doesn't supply an object manager, but it does provide a few more events for an Object Manager to receive than raw DGD does.

The Kernellib does track numbers of active objects and allows setting quotas for resource usage.

## Resource Management

The Kernellib's counting of resources extends to allowing quotas and limits on them. That can prevent having too many object instances active at a time, or allow setting limits on number of "ticks" (CPU usage) or callouts.

It also has a simple system for creating and managing users themselves, naturally. And it provides a simple structure for managing network ports and the attached "user" and "connection" objects for network I/O.

## Inheritance and Object Life Cycle

Kernellib makes sure that if an object is inherited ***from***, it's a parent object whose path contains /lib/ somewhere. It makes sure if it's ***inheriting*** then it's ***not*** a parent object.

That way, parent objects will only exist as part of child objects and will never have their own state that needs to be carefully saved. They are state-free and can be recompiled (and if necessary, destroyed) freely without disrupting any other objects.

With an object manager, you can also set callbacks on objects to make them respond appropriately to being compiled, instantiated, updated or destroyed.

## A Simple UI

The Kernellib includes a very simple 'wiztool' - a set of commands for administrators of your application to query and manage objects. You'll want far more than this in the long term, but you'll be glad to have it in the beginning, and likely later as well.

## Future-Proofing

The Kernel Library provides a standard base for building. If DGD changes, the Kernel Library can help shield you. For instance, when the behavior of send_message changed several years ago, the Kernel Library changed its behavior so that all Kernel-based MUD Libraries still worked on new versions. The Kernel's abstraction layer can stay standard, even across different versions of DGD.

The Kernel provides some future-compatibility — it uses thread-local storage in several places that you might not know to, so every Kernel-based MUD Library is more likely to work well with the upcoming multiprocessor version of DGD. You'll still need to fine-tune, but the Kernel gives the basics for free.

By keeping your programming on the straight and narrow, the Kernel library is also handling needs you don't (yet) know you have. For instance, you may not need to upgrade your objects initially, but if you use the Kernel then you can be certain that if that need comes up, you can meet it. Similarly, you may initially have only a single administrator on your MUD. If you then need to add a second or third administrator, the Kernel's built-in security makes it much easier to keep them from intentionally or negligently causing problems. Since the Kernel Library was designed with very large projects in mind, and has been used in very complex projects, you can be sure that it's suitable for them. Your own homebrew solution may require more work to fix the same problems.
