@@TITLE Kernel Overhead@@

<h2>Overhead and the Kernel Library</h2>
<pre>
From: DGD Mailing List (Felix A. Croes)
Date: Sat Feb 21 04:41:01 2004
Subject: [DGD] Resolve Path & Array Efficiency

"Steve Foley" wrote:

>[...]
> When I think of the normalize_path function, I think the major inefficiency that
> could crop up is memory allocation (caused by new arrays).  Felix's
> normalize_path basically minimizes the number of new arrays, and I honestly
> couldn't write a more efficient algorithm (but that isn't saying a whole lot).
> But you're right, it's not real easy to follow.

The kernel library imposes some overhead.  I've done by best to make
this overhead as small as possible, while still cramming in all the
functionality that I want.  Most of the code is written for speed.

DGD's memory allocation is very efficient (with profiling using a
2.4.5-based mudlib, memory management measures about 2% of total
execution time), but creating the smallest amount of intermediate
values that require memory allocations -- strings, arrays, mappings
and objects -- is still a good stragegy.

Regards,
Dworkin
</pre>
<hr>
<pre>
From: dgd at list.imaginary.com (Noah Gibbs)
Date: Tue Jan  6 16:45:01 2004
Subject: [DGD] A simple lib

  This is old, but I meant to reply.

--- Robert Forshaw wrote:
> I was most detered from the kernel lib because
> it looked like it would 
> consume too much resources. My MUD will be
> making heavy use of very short 
> (milisecond) callouts and I don't want it
> lagging because the resource 
> manager itself is hogging the CPU. Am I wrong
> in this assessment?

  Possibly.  I've never had any performance problems
with it, but I'm also doing something that acts a lot
like a regular MUD.  Since a regular MUD uses an order
of magnitude less CPU than a modern desktop machine
provides (if that!), the small amount of waste that
the Kernel Library involves is simply not an issue.

  If you're going to do something that requires nearly
your entire CPU load, and you're going to do it in
tiny amounts at millisecond precision, then security
and sanity-checking both need to go.  They're too
expensive.  So you can write a less-secure and
less-debuggable version than the Kernel, which will be
faster.

  Of course, if you're doing something that
performance-intensive and precise, why are you using
LPC?
</pre>
