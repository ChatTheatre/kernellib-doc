---
title: Breaking up Functions
layout: default
---

(NOTE: DGD's "threading" here is not the same thing as "threading" in other contexts. I don't know a standard name for this in Node.js, though it definitely has the same thing. Non-evented platforms often have no direct parallel to this.)

## Threading and Long-Duration Functions

You probably already know that DGD is a disk-based MUD, which
means that it'll almost always keep your stuff swapped out when not
in use. It also means that in general you can't iterate through
every object since that would drag more objects into RAM than you
have RAM -- a definite no-no.

Fair enough. Nothing's wrong with that. But how does the ObjectD
do its thing? Doesn't it need to go through every object in order
to recompile the AUTO object?

It does. And for larger MUDs, you can solve the problem above
<i>and</i> the problem of long operations that take more ticks than
can fit in a single thread in a single, pretty simple way. If you
already know about call_outs, then you already know the basic
building block of the approach.

The basic idea is that you split the operation into a series of
steps, rather as if you were putting it into a loop. The important
thing about the steps is that they take less than a single thread's
worth of ticks, and touch far less than every object in memory --
at least, individually. All together they can do anything you want.
Remember, DGD will swap objects in when you use them and will swap
them out when it runs out of space -- but only at the end of a
thread. You're going to put each of these steps you divide your
function into in its own thread using call_out.

In fact, you can put it into multiple helper functions. For
security you'll want to make them static. For call_out to work, you
<i>can't</i> make them private, so don't (AUTO has to call them, at
least in the kernel MUDLib). For many functions, that's basically
enough and it'll all work fine. But it's still not quite the same
as doing it all in one go.

The biggest missing component is that if you just do your
function in a series of call_outs, you can be interrupted
<i>halfway through</i>. Even if you set the call_out delay to zero,
which you'll want to, somebody else can still call your object when
you're only halfway done. For simple objects you'll just need to
grin and bear it -- make sure your object can deal with these
calls.

But for big, vital, mission-critical components that's just not
acceptable. For an objectd, for instance, you don't want to have
your MUD keep churning while only <i>half the objects are
upgraded</i>. That's just inviting disaster, and it certainly makes
for a very unpredictable system. So instead, you need a way to lock
down the rest of the system and keep it from running.

The Kernel MUDLib's rsrcd gives you most of what you need
through the suspend_system call, which you (naturally) must be
privileged System code to use (in the ~System directory, usually).
You can reverse it with release_system() when you're done, or in
<i>any error case</i> -- you don't want to leave the system
suspended if you only get halfway done. Incidentally, check the
return value from call_out. If it returns a number less than one,
that means an error of some kind occurred and your new call_out
isn't scheduled. Be sure to release the system's call_outs in that
case too.

That's fine (you can see Geir Harald Hansen's objectd (TODO: LINK) for a
great example of how all of this works, by the way), but it doesn't
fully solve the problem. You need to keep any new threads from
starting until you're done. That means stopping other people's
call_outs as described above. But it also means keeping new
connections from happening or old connections from getting new
input. And for that, you'll need to do the work.

The work, in this case, consists of messing with the telnetd and
the user objects. TelnetD can turn away connections with an error
message -- just return -1 when querying the delay and the
connection will close immediately. Remember, you don't want to do
anything complicated because this code can run when your MUD is
only half-recompiled!

For user objects, you'll probably want to send a message to each
connection <i>before</i> suspending the system and then have them
all ignore all input while it's suspended. You can query userd for
a list of all open connections. See why you usually just want to
let it be possible to call your object when you're halfway done?
However, for major things like recompiling the AUTO object there's
just no other good way to do it. You can recompile by the seat of
your pants (as Phantasmal does as of this writing, April 8th,
2002), but at some point it'll come back and bite you.

Since suspending all these things (call_outs through rsrcd,
connections through telnetd, input through the user objects) is
kind of a pain and involves touching a lot of objects, you might be
tempted to write a simple library function to do it. Bear in mind
that when you suspend call_outs through rsrcd, it uses
previous_object() to check its caller and suspends call_outs for
<i>everyone but that caller</i>. That means that the library you
put the call in can register call_outs, but you, who called it,
cannot. That almost always means your call_outs won't happen and
neither will anybody else's. The library probably registers no
call_outs of its own. The other work (telnetd, user objects) can be
put into a library safely if you write the code reasonably.

There's not yet useful code for all of this in this article. I
hope to remedy that in the future. In the mean time, <i>do</i> look
at Geir Harald Hansen's
objectd for an excellent demonstration of many of the
principles involved.
