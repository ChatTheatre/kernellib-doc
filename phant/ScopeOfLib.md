---
title: What Does the Kernel Library Actually Do?
---

DGD is able to fully recompile all code in place, if you meet
certain restrictions. The Kernel Library, used properly, makes sure
that you obey those restrictions. There's no way to violate them,
in fact, if you don't modify anything under /kernel, because
the Kernel Library simply won't allow you to. So the Kernel Library
helps make sure that all your code can be upgraded instantly and
in-place.

The Kernel Library also provides security. If you've got code
and data files in your directories, you'd like it if other
administrators and builders can't change them, and for some of the
files, you don't want other admins (or their objects and programs)
to be able to read them. The Kernel Library prevents that, not only
for your files but also for your compiled programs running in
memory. That means you don't need to worry as much about trusting
other administrators in the game, since they can't mess up your
stuff.

The Kernel Library also provides certain kinds of object
management support and user management support. You can register an
object manager with the library, and the Kernel Library will call
hook functions in your object manager whenever files are compiled,
inherited or included so that you can do the rest of the work
necessary to upgrade every object in place (you can also use
somebody else's object manager, but it still needs the Kernel
Library). The Kernel Library will keep track of user connections
and disconnections, and provide you with a nice interface to it.
There are similar facilities to track errors, and cause all
programs to implicitly inherit an AUTO-type object of your
choice.

The Kernel Library will also keep track of resource usage for
every administrator in the game. The amount of memory space and
processor time and the number of DGD objects are tracked
automatically, as are several more obscure commodities. It's easy
to add more resource tracking for things like in-game money or
power of monsters to keep your administrators and their areas from
spawning too much of a good thing.

There are some additional functions defined by the Kernel
Library for creating signals which an object can send and other
objects can subscribe to. That allows for various events occurring,
and provides an infrastructure to make it happen in code.
<pre>
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
</pre>
<hr>
<pre>
Date: Mon Sep 17 12:51:00 2001
Subject: [DGD] Could it work...

Matt,

> To be honest we looked at the kernel lib and had no idea what was going on
> ;) So with a lot of work and head banging we built our own system from the
> ground up. The great thing about that is that I know what every little bit
> of code does. I was unsure about working with something I couldn't
> understand. Maybe now we can look back and say ... 'oops' but I think it's
> okay...


The kernel library is not particularly friendly, true, and I completely 
agree that it is good to learn why you need something before you start 
using it. To limit the amount of "oops" that happens, seriously consider 
my suggestion not to start off with a persistent Mud. Then over time as 
you run the Mud, take note of how often you need to cold-start it and 
consider what would happen if you couldn't do that, ever. That'll give 
you a first appreciation of what need the kernel library attempts to fill.

Zell
</pre>
<hr>
<pre>
Date: Tue Sep 18 19:25:01 2001
Subject: [DGD] Could it work...

On Wed, Sep 19, 2001 at 12:22:35AM +0100, mtaylor wrote:
> Me again I'm afraid,
> 
> My coding partner thinks that we can use what we need of the kernel lib to
> develop our own system. The good thing for us about doing that is that we
> will then understand what is going on.
> 
> Does anyone have the time (and/or patience) to list what it is the kernel
> lib does and very briefly how and why it does it.

Disclaimer: I doubt this list is complete.

The kernel-lib...

- provides the basic ability to add users, grant users access, grant
  users limited quota of objects, stack/ticks/callout-usage, to avoid
  someone going ahead and creating 60000 clones of some object and
  thereby taking the game down.

- provides the ability for external objects to 'hook' themselves into
  the /kernel/sys/driver.c object (and one or two other /kernel
  objects) so that you will be notified if an object is compiled,
  cloned, destructed, freed, inherited, a file #include'd, etc.

  This is where you would hook up an 'Object DB' that keeps track of
  the inheritance information so that you can write your own 'upgrade'
  command that recompiles whatever is necessary.  This is one of the
  things that is _not_ provided by the kernel-lib. :)

- provides a basic wiztool (with a very nice 'store results for reuse'
  approach).

- enforces that inheritables, clonables and 'other' code are in
  separate directories, 'lib/' subdirs for inherities, 'obj/' subdirs
  for clonables and, more recently, 'data/' subdirs for LWOs.

I'm sure there are other useful things but these are just some that
come to mind right now.

Hope this helps,

Erwin.
-- 
Erwin Harte
</pre>
