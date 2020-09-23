---
title: Kernellib Miscellaneous
layout: default
---


### Why Wouldn't You Use It

Some people prefer to write their own code, just in general. That's usually a very, very poor reason to avoid the Kernel Library — it solves many subtle problems, and you really need to understand them thoroughly if you don't use the Kernel. Thread-local storage, object upgrading, information being passed insecurely through the call-stack... These are all topics that need to be addressed, but for which there is no good documentation. The Kernel solves each of them, and many more.

A more common reason to avoid it is that it seems too complex. The Kernel Library seems to give very little until you understand it well, and there's definitely a lot of code there. However, as this page should demonstrate, the Kernel Library addresses a lot of different concerns, and it does so quite well. It's complicated, but usually only because it has to be complicated. If there's something you're not concerned with that the Kernel Library does (like security, say), you're probably better off tearing the security-related code out of the Kernel Library and using the result than writing your own replacement... The Kernel does a ridiculous variety of stuff, and it will be very hard for you to make sure you do all the same things.

Another common complaint is that the layout is unfamiliar. The Kernel Library has an unusual security model based on objects checking what program is calling them. It handles significant security and data integrity issues by checking the pathname of the program, so moving a piece of code from one directory to another (or calling it from a different function or object) can significantly change the behavior. The security model can be both powerful and elegant once you understand it, but it requires significant effort to understand. This is a significant issue — requiring MUD Library authors to understand unfamiliar code has held DGD back for years, and it's hard to find builders and coders who already know LPC, let alone DGD's Kernel Library. However, the only solution to the issue is to turn your MUD into yet another LPMUD clone. If you're doing something new and innovative, you're going to have to work with some unfamiliar code, and so are your builders. If you're *not* doing something new and innovative, why are you bothering to write your own MUD Library?

## The Kernellib as "Complexity"

```
From: dgd at list.imaginary.com (Par Winzell)
Date: Mon Sep 17 12:39:01 2001
Subject: [DGD] Could it work...

>>I'm sure I've missed loads but I wasn't entirely sure why the kernel lib was
>>so important to use. The above does seem tricky but doesn't sound
>>impossible. With looking at the kernel I'm sure we can work it out seeing as
>>that's how we've coded the connection stuff etc. so far.
>>Or am I being dim?
>>
> I think the point people are trying to make is that very careful thought
> has gone into making the kernel lib, and by using the kernel lib, you are
> saved from doing a lot of trial-and-error attempts at your own 'kernel
> lib'.  Especially if one is not terribly interested in the knitty-gritty
> details, that's where I'd assume the kernel lib helps; at least that's the
> point I think Par was


That's pretty much it. You have to ask yourself, how much of what the
kernel library gives me do I need? And if that amount is large enough,
then you need to question if it is realistic to believe you can -save-
effort by -not- using it.

If you are going for a truly persistent Mud, i.e. you plan on rebooting
only from state dumps once your Mud is up and running, then you -have-
to either use the kernel library, or reimplement much of what it does.
That is a non-trivial undertaking, to say the least.

Going persistent is a big deal. When you're persistent, you realize, you
will never again cold-start your game. You never again get to feel like
you're cleaning away all the old cobwebs. If you clone an object and you
lose track of the clone, it's going to stay with you for eternity. If
you compile a program and forget where it is, it'll stay compiled and
forgotten forever. Crap builds up. If you make a terrible error a year
into your Mud's uptime, an error that causes you not to be able to log
in, you're screwed -- you have to go back to a saved statedump from a
few days earlier, or if you didn't save any, lose all your state.

If Matt wants to avoid all this complexity, he should not try for the
persistence right now. It cannot be overstated what a difference this
makes. Save inventories and vital player data to file, perhaps using the
trusy old save_object() and restore_object(). Worry about persistence in
a few years. You really don't want to go for persistence as your first
project.

Zell
```

## The Kernel Library and Persistence

There was an e-mail exchange between Frank Schmidt and Dworkin about the suitability of the Kernel Library as a general-use MUD Library. Some of it is excerpted below.

```
> The main idea is that it requires mudlib code ontop of it, I haven't said
> anything else. But the kernel lib is far from what I see 'fit' for MUD
> programmers world wide, when lacking lots of "general" functionality to
> handle arrays, strings, mappings, math, sorting algorithms, etc, etc,
> which in my opinion belongs in the auto object. (Previous mailinglists
> explains why) And that's just one of the issues, each time you need
> something special (which you know DGD can offer), a General kernel lib
> will probably not support it.

Your view of what a kernel library should be seems to agree perfectly
with the function of the objects in the /dgd directory tree in the
2.4.5 mudlib -- which is certainly not that of DGD's kernel library.
Beyond that, I think you also fail to understand what the kernel
library can do, as evinced by your earlier comment that it "occupies"
the auto object and driver object.  If there is one thing that the
kernel library is good at, it is modifying or completely overriding
the behaviour of those two objects.

I think that some of this blindness is caused by the extraordinary
success of LPmud 2.4.5.  To get beyond that, let's take a look at
a completely different mud, Ultima Online, comparing features with
those of traditional LPmuds:

 - UO is persistent.

   Persistence is DGD's most important single feature (I like the
   term "continuous mud" better, but "persistent mud" is the
   standard term these days).  A persistent mud needs a design
   radically different from that of a traditional LPmud:

    - There has to be a way to change the behaviour of existing
      objects.  The kernel library is designed in such a way that
      upgrading objects -- that is, recompiling them without first
      destructing them -- is possible for all objects, given the
      limitations imposed by LPC inheritance.
    - If you have guest-programmer wizards like traditional
      LPmuds, you need a way to limit the resources available to
      individual wizards, since rebooting the mud to get rid of
      undesirable objects is not an option.  The kernel library
      has a generic resource management system which manages
      such things as number of objects or number of callouts by
      default, and to which new mudlib-specific resources can be
      added at will.
    - Such functionality as string formatting should not be in the
      auto object.  Having to recompile the 3D space code because
      a change was made in string formatting is ridiculous.

 - UO has a custom client.

   The kernel library cannot make any assumptions about what sort
   of client is being used.  It cannot even assume that everyone
   uses the same client.  It merely attempts to be as little in
   the way as possible -- not only in the matter of communications.

 - UO has no traditional rooms, add_actions, etc.

   All such things have no place in the kernel library's auto object.
   Similarly, nothing that is not needed in  muds within its
   target range has a place in the kernel library.
```
