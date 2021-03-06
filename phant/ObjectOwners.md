---
title: Object Creators and Owners
layout: default
---

## What is an Object's Creator?

Finding an object's creator is easy. The object's name will
usually start with /usr/XXX. That XXX is the object's creator as
far as the Kernel MUDLib is concerned.

## Who Owns the Object?</h3>

The Kernel MUDLib occasionally does things with objects
according to who owns them. The owner is, conceptually, the user
who called clone_object() or the equivalent. But it's a little
trickier than that.

Say you've just entered a command which causes an object to be
cloned. You're the current user, but you're using code in (for
instance) /usr/System to run the command. So who owns the new
object, you or System?

Say it's not a clone? Does that make a difference? Yup. If an
object isn't a clone then the creator is also the owner. For
instance, that's true if it's the original compiled clonable
object, or if it's a library.

<pre>
From: dgd at list.imaginary.com (Par Winzell)
Date: Fri Feb 27 13:06:01 2004
Subject: [DGD] Kernel Lib, creator & owner

>>So...are there any instances where the variables owner & creator in the
>>auto.c will actually be different? or the driver's creator() return a
>>differing string? ... and if not why have 2 variables that don't differ?
>>
>>I'm sure I'm missing something, so could anyone enlighten me? :)
>>
> 
> Well I tested some more, logged in as admin.
> copied, blah.c to /usr/zamadhi/obj/blah.c
> compiled it, cloned it.
> The master object winds up with zamadhi as creator & owner.
> And the clone winds up with "admin" as owner, and zamadhi as query_creator()

You can easily scan through auto.c for all instances of owner and 
creator being assigned anything. It all happens in _F_create and the 
logic is quite straight-forward:

   * The creator is purely a function of the object's name...
     * An object /usr/Foo/* has creator Foo
     * An object /kernel/* has creator System
       * (i.e. /kernel and /usr/System have some equivalences)
     * Anything else has creator nil

   * The owner is identical to the creator for master objects (anything 
that is neither clone nor pure program). For clones it is set to the 
owner of the cloning object, unless the second argument to 
clone_object() and new_object() is used by the mudlib layer to force a 
certain owner for an object. This is how a wiztool that lives in /kernel 
can give a newly cloned object e.g. the owner 'zell'.

The clone_object() and new_object() functions pass on this 'uid to be' 
through a TLS value (thread local storage).

Zell
</pre>
