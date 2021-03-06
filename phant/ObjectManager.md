---
title: Object Manager
layout: default
---

There's a lot in the mailing list archives about how to write an
Object Manager. Since you're better off adapting mine or Geir
Harald Hansen's, I'm just going to collect general information for
any other masochists who want to write their own. That information
follows.

<pre>
Date: Mon, 21 Jun 1999 07:59:51 -0700 (PDT)
From: Par Winzell
To: DGD Mailing List
Subject: [DGD]upgrading kernel library

Geir,

You're on the right track, and you're in luck. Thanks to the kernel
lib enforcing the total separation between inheritable and stateful
code, it's easier than it might be to write this kidn of automatic
upgrading system.

So, you want to upgrade /foo/lib/bar, which is inherited by a bunch
of other programs, possibly including some destructed ones that are
not yet freed (due to lingering dependencies). You can't recompile
it because it's inherited, so you have to destruct it.

To completely upgrade /foo/lib/bar, you need to observe the tree of
all programs that depend on it. All the interior nodes of this tree
are stateless inheritables, and so can be destructed as well. Next,
you step through all the leaf node objects -- clones, daemons, etc.

As you recompile them, one by one, you remove dependencies from the
old issue of /foo/lib/bar until finally the last recompile frees
the last dependency -- and the issue is freed.

Things to note: Unless you forbid destruction of a program that has
dependencies, you do have to use the ID's rather than just the path
names in the database. If such destruction is possible, there will
be more than one issue of the same program in memory at once (even
if only one is visible from the LPC layer), and you definitely need
to keep separate records for these issues!

Also, for a real-sized Mud, you need a datastructure that can hold
more than ARRAYSIZE dependencies. If you switch to using ID's, that
is probably solved most easily with an array of mappings, something
like issue = map[id>>8][id%0xFF].

One complex issue is how to deal with upgrading e.g the auto object
in a fully-grown Mud, where the swap file can be hugely larger than
the RAM in the machine. Recompiling the leaf objects in one single
thread becomes unworkable; DGD must be given the chance to swap out
now and then. This essentially requires basic support for freezing
all activity in the lib, explaining the kernel lib's support for
call_out suspension.

Zell
</pre>
<hr>
<pre>
Date: Tue, 22 Jun 1999 17:31:02 +0200 (CEST)
From: "Felix A. Croes"
To: DGD Mailing List
Subject: Re: [DGD]upgrading kernel library

Rasmus H Hansen wrote:

> This upgrading of base objects seems to be a candidate for a kernel library
> add-on. Immediately I don't see any need for great variation on how this is
> implemented - but then again I haven't coded this, I merely thought it
> over. Would anybody care to share such an implementation or maybe Dworkin
> can be persuaded to include automatic rebuild (or whatever this feature
> should be dubbed) in a future version of the kernel library.

I know of three existing implementations of global upgrading, and
they're all substantially different.  Only two of them use the
kernel library.  I see room for more variants.

The kernel library doesn't include this because it is supposed to be
minimal.  It does provide all the hooks to make it possible, though --
see the object manager documentation in dgd/mud/doc/kernel/hook. I
estimate that it can be implemented in about a thousand lines of
code.


> (Fear the rollback of an atomic rebuild.)

Compared to the time it took to do the global upgrade, a rollback
due to an error in atomically executed code will be almost
unnoticable.

Regards,
Dworkin
</pre>
<hr>
<pre>
Date: Thu, 1 Jul 1999 00:22:32 +0200 (CEST)
From: "Felix A. Croes"
To: DGD Mailing List
Subject: Re: [DGD]upgrading kernel library

Two in one.

Geir Harald Hansen wrote:

> > > Dworkin wrote:
> > Possible variants:
> > 
> >  - keep track of different issues of compiled objects, including their
> >    sources.
> >  - don't keep track of different issues; instead, ensure that only
> >    one issue exists of an object at any time by making the upgrade
> >    operation atomic.  Disallow destructing of lib objects that are
> >    still inherited other than by the upgrade operation.
>
> How can I make the upgrade operation atomic if I need to split it over
> several callouts to permit DGD to swap out objects during the upgrade?
> The recompilation of a non-inheritable could fail and leave separate
> versions of inheritables in existence.

You can't.  You would have to do it all on one thread to make the
operation atomic.


> Another thing, I suspend callouts.. Should I also somehow block all user
> input and block new logins while upgrading?  How?

You can block new connections by temporarily setting the login
timeout to -1.  You can block input on connections using
block_input() in /kernel/lib/connection.c.


Stephen Schmidt wrote:

>[...]
> 1) I gather that by "global upgrade" we mean that, when we
> update object A, we also want to update all objects B that
> inherit A, and probably update all clones of A and B also.
> Since this is not default behavior (by default, A's clones,
> B, and B's clones would keep the pre-upgrade behavior until
> they were reloaded) the mudlib has to keep track of everything
> that inherits A and systematically replace old code with
> new code. Is that about right, or have I missed a step?

You may not realize that with "upgrading" I mean something other
than "updating".  Global upgrading is the process of recompiling
a set of objects without destructing them first, allowing them
to retain state in the form of variables and callouts.  Of all
the LPC servers, only DGD has this capability.

However, it can only do so for one object (and all of its clones)
at a time, and only if the object is not inherited by another.
The division between inherited objects and other objects, enforced
by the kernel library, ensures that there always is a way to
upgrade any single object.  Global upgrading must coordinate
the process for a set of objects, usually all those that inherit
some specific object or objects.

Regards,
Dworkin
</pre>
<hr>
<pre>
Date: Tue, 6 Jul 1999 21:38:56 +0200 (CEST)
From: "Felix A. Croes"
To: DGD Mailing List
Subject: Re: [DGD]upgrading old clones

Geir Harald Hansen wrote:

> I just had an idea, and it is not well thought through yet, but I thought
> I'd throw it in here and we'll find out whether it is good or not. ;)
>
> I think sometimes a kfun to upgrade clones of an old issue may be useful.
> It need only work when there is a newer issue whose master object is
> not destructed.  The older issue may not have any inheriting dependents,
> but has at least one cloned dependent, of course, or it would not exist.
>
>    object upgrade_clones(int issue_id)
>
> This function would upgrade all clones with that issue ID to the newest
> issue, adding to its number of clones.  It returns the new master object
> issue of the clones, which already exists before this function call, or nil
> if it fails somehow.  When this call succeeds, the old issue is removed.
> Hmm, maybe it would be better to return the number of clones upgraded.

It is arguably wrong for a cloneable object ever to have different
issues; after all, shouldn't all clones have the same behaviour?
Therefore I prefer simply to make destructing an object with clones
impossible.

Regards,
Dworkin
</pre>
<hr>
<pre>
Date: Thu, 6 Jan 2000 03:21:13 +0100 (CET)
From: "Felix A. Croes"
To: DGD Mailing List
Subject: Re: [DGD]Object Upgrading Scenario

> From: "Jason Cone" wrote:

>[...]
> Anyway, let assume the following object relationships:
>
>     A -i-> B -i-> C -c-> D
>     A -i-> B -i-> C -i-> E -c-> F
>
> This notation should read, "Object D is a clone of object C which inherits
> B, which inherits A.  Object F is a clone of object E which inherits object
> C."
>
> Bad design issues aside, what is a possible implementation that could take
> object E into account?  If E (and, consequently, F) didn't exist, an
> overloaded compile_object() function would do the following:
>
>     destruct_object(A)
>     destruct_object(B)
>     ::compile_object(C)
>
> This would then allow for the single/multiple D object(s) to take advantage
> of the new functionality that was added to A.  That approach, however, would
> really mess things up if it used the same approach when trying to upgrade E
> (F) -- ::compile_object() could not be called on object C as E inherits C
> (would yield a run-time error).
>
> Thoughts?

There isn't any way to upgrade all existing objects in your scenario.
It is a special case of the more general problem:

    If an object is both inherited and used in any other way (either
    it has had a function called in it, or it has been cloned),
    upgrading both this object and those that inherit it is not
    possible.

This is precisely why I prevented using/cloning of inheritable objects
in the kernel library.


> I know this subject has been beaten to death, but since we're running across
> this issue during the implementation stage, my hope is that we can share a
> successful implementation with everyone and not just discuss it on a
> hypothetical issue.  I would like this scenario resolved, though, if
> possible. :)

With the existing primitive operations (compiling an object, recompiling
an object, destructing an object), upgrading of all objects is not
possible in your scenario.

By the way, the subject is far from beaten to death.  I rather think
it died prematurely. :)

Regards,
Dworkin
</pre>
<hr>
<pre>
Date: Thu, 6 Jan 2000 04:30:01 +0100 (CET)
From: "Felix A. Croes"
To: DGD Mailing List
Subject: Re: [DGD]Object Upgrading Scenario

Neil McBride wrote:

>[...]
> > There isn't any way to upgrade all existing objects in your scenario.
> > It is a special case of the more general problem:
> > 
> >     If an object is both inherited and used in any other way (either
> >     it has had a function called in it, or it has been cloned),
> >     upgrading both this object and those that inherit it is not
> >     possible.
> > 
> > This is precisely why I prevented using/cloning of inheritable objects
> > in the kernel library.
>
> So, was it intentionaly designed this way in the driver, or is it simply
> something that couldn't be worked around? (ie - the master keeps track
> of clones and desting it loses track of them perhaps - I don't know ;) 

I looked long and hard at implementing everything in the server
back then, but eventually I decided to implement only minimal
upgrading support (recompiling a single object) because the
code would have been enormously complex at the server level.
It is fairly simple in LPC (about 1000 lines if you start with
the kernel library).  Given that this was completely new
functionality which nobody had ever asked for and which many
even failed to see a use for, I felt that it was not unreasonable
to put some restrictions on mudlibs that wanted to have it.

Another reason is that an LPC implementation -- as usual -- is
more flexible.  For example, suppose you upgrade the auto object
and everything that depends on it.  Recompiling every single
object in the mud in one go would cause an enormous amount of lag,
as well as consume a large amount of memory.  In LPC, the operation
can be broken into several pieces, with other tasks + regula
swapping in between.


> The way I see it is that being able to make an instance of an object
> (ie, clone) _and_ inheriting the original object is something that
> should be possible.  I don't really understand why it's considered a bad
> design, except in relation to the way DGD treats inheriting/compiling.

It all depends on whether you like the separation between class
(inheritable) and object.  If you do, then anything that ignores
that difference is bad design.


> Also, is there some reason as to why the extra functionality the kernel
> lib provides is not made a standard part of the driver itself

Backward compatibility.  Also, not everyone appreciates the distinction
between class and object.


> - possibly
> along with the changes to make global upgrading completely automatic
> (except for the outlined scenario ;) ??

I understand that you'd rather have me implement it in the server
than do it yourself, but I hope that the above clarifies my reasons
for not having done so.

Whatever came of the idea, put forward on this list, to implement
global upgrades in LPC and make the code freely available?

Regards,
Dworkin
</pre>
<hr>
<pre>
From: Neil McBride
To: DGD Mailing List
Subject: Re: [DGD]Object Upgrading Scenario

> I understand that you'd rather have me implement it in the server
> than do it yourself, but I hope that the above clarifies my reasons
> for not having done so.
> 
> Whatever came of the idea, put forward on this list, to implement
> global upgrades in LPC and make the code freely available?

Back to the original problem then.  The proposed scenario is

     A -i-> B -i-> C -c-> D
     A -i-> B -i-> C -i-> E -c-> F

The problem here is when D clones C.  The solution is an adaptation of a
solution I 'thought' I'd worked out a few days ago, until I realised my
memory of how it all worked was a little backwards ;)  Anyway, for ease
of explanation, we'll call C the primary master, a new object, CX,
called the secondary master and D will be the clone

Now, when a call is made to clone an object, the clone_object can create
the secondary master by creating a file that inherits the primary
master, and nothing else.  The name of this can be determined by some
algorithm so they can all be hidden from the user, thus making it all
transparent.  The clone D, can then be made of the secondary master,
leaving the primary master free to be inherited.

This will also fit right into Jason's original ideas on how to deal with
upgrading inherited objects.  Obviously, you would not hide these files
from the internal processes dealing with it all ;)

However, I'm not sure how to deal with the calling of functions in a
master.  I imagine we could overload call_other as I think the
{ob|file}->fn call uses call_other.  Correct me if I'm wrong.  I think
someone with some more knowledge of how the function calling works needs
to finish this off ;)

Cheers,

Neil.
</pre>
<hr>
<pre>
From: John "West" McKenna
Subject: [DGD]upgrading
To: DGD Mailing List
Date: Mon, 10 Jan 2000 22:45:31 +0800 (WST)

There's been a bit of talk about upgrading objects, and there's always
been people wanting code for it.  So...  This is how I'm doing upgrading
of objects.  Like Dworkin's kernel lib, I distinguish between classes
and objects.  Actually, I make a three-way distinction:

  1) libs: Anything with /lib/ in its path can be inherited.  It cannot
     be cloned.  No other objects can be inherited.  create() is not
     called, and the blueprint should not be used.
  2) clones: Anything with /obj/ in its path can be cloned.  No other
     objects can be cloned.  create() is called on the clones, but not
     the blueprint.  The blueprint should not be used.
  3) singles: All other objects have create() called on the blueprint.

2 and 3 together are called 'objects'.

If you're foolish enough to have *both* /lib/ and /obj/ in your path,
you're treated as a lib.

There is also mention of 'physical' objects.  These are clones that have
some 'real' existence in the MUD universe (including players).  Any
clone that wants a real existence inherits /kernel/lib/physical, which
defines (among many other things) the function is_physical() to simply
return 1.

The code below is NOT the best way of doing things.  In particular, I'm
not at all happy with the way that I'm including the auto object in the
inheritence table.  There's a bug in there somewhere too - it fails in
interesting ways if there was a compilation error.  

Hopefully having some code will provoke a little discussion, and maybe
even a little code...

/*  The auto object */

int is_clone() {
  return (sscanf(object_name(this_object()), "%*s#"));
}

void _create() {
  /* create() gets called for clones of objects in a .../obj/... directory, and
     for blueprints of objects that are not in a .../obj/... or .../lib/...
     directory, but not for anything else.  In particular, inherited objects
     and the blueprints of clones don't get create() called. */
  string name;

  name=object_name(this_object());
  /* It can only be a clone if it was in a .../obj/... directory, so no need
     to explicitly check for that */
  if (sscanf(name, "%*s#")) {
    this_object()->create();
  } else {
    if (!sscanf(name, "%*s/lib/") && !sscanf(name, "%*s/obj/")) {
      this_object()->create();
    }
  }
}

object compile_object(string name) {
  object o;

  o=::compile_object(name);
  /* Make sure create() gets called */
  call_other(o, "???");
  return o;
}

object clone_object(object obj) {
  if (sscanf(object_name(obj), "%*s/lib/")) {
    DRIVER->message("Illegal clone: "+mixed_to_string(this_object())+
      " attempting to clone "+mixed_to_string(obj));
    return nil;
  }
  if (!sscanf(object_name(obj), "%*s/obj/")) {
    DRIVER->message("Illegal clone: "+mixed_to_string(this_object())+
      " attempting to clone "+mixed_to_string(obj));
    return nil;
  }
  return ::clone_object(obj);
}

/* The driver object */

mapping inheritence;
mapping upgrading;
mapping last_upgraded;
int reboot_time;
string *upgraded_clones;

void initialize() {
  /* ... */
  reboot_time=time();
}

object inherit_program(string from, string path, int priv) {
  if (!sscanf(path, "%*s/lib/")) {
    message("Illegal inherit: "+from+" attempting to inherit "+path);
    return nil;
  } else {
    if (!inheritence) {
      inheritence=([ ]);
    }
    if (!inheritence[path]) {
      inheritence[path]=({ });
    }
    if (!inheritence[AUTO]) {
      inheritence[AUTO]=({ });
    }
    /* !!! This is really ugly.  There must be a better way of doing this.
       But we're not told when someone inherits AUTO.  So we just add it
       whenever any object gets mentioned (because if the object exists,
       it inherits AUTO). */
    inheritence[AUTO]=(inheritence[AUTO]-({ from }))+({ from });
    inheritence[AUTO]=(inheritence[AUTO]-({ path }))+({ path });
    inheritence[path]=(inheritence[path]-({ from }))+({ from });
    return load(path);
  }
}

static string *_upgrade(string path) {
  string *inherited_by;
  object obj;
  string *done;
  int i;

  done=({ });
  if (inheritence) {
    obj=find_object(path);
    inherited_by=inheritence[path];
    if (!upgrading[path]) {
      upgrading[path]=1;
      if (inherited_by) {
        /* It's a lib - destruct it and upgrade everything that inherits it */
        last_upgraded[path]=time();
        destruct_object(obj);
        done+=({path});
        for (i=0;i<sizeof(inherited_by);i++) {
          done+=_upgrade(inherited_by[i]);
        }
      } else {
        /* It's an object - if a blueprint exists, recompile it.  If there is
           no blueprint, there's no point - it'll get compiled next time it
           is used. */
        if (obj) {
          last_upgraded[path]=time();
          compile_object(path);
          upgraded_clones+=({path});
          done+=({path});
        }
      }
    }
  }
  return done;
}

static string *auto_upgrade(string path) {
  string *inherited_by;
  mixed *dir_info;
  int last_upgrade, modified_time;
  int i;
  string *done;

  done=({ });
  if (last_upgraded[path]) {
    last_upgrade=last_upgraded[path];
  } else {
    last_upgrade=reboot_time;
  }
  dir_info=get_dir(path+".c");
  if (sizeof(dir_info[2])) {
    modified_time=dir_info[2][0];
    if (modified_time>last_upgrade) {
      done+=_upgrade(path);
    }
  }
  inherited_by=inheritence[path];
  if (inherited_by) {
    for (i=0;i<sizeof(inherited_by);i++) {
      done+=auto_upgrade(inherited_by[i]);
    }
  }
  return done;
}

string *upgrade(string path) {
  /* If given a path, upgrade that object and everything that depends
     on it.  If not given a path, do an automatic upgrade of every file
     that has been modified since it was last recompiled.
     Returns an array of the paths of all objects that were recompiled. */
  string *done;

  upgraded_clones=({ });
  upgrading=([ ]);
  if (!last_upgraded) {
    last_upgraded=([ ]);
  }
  if (path) {
    /* Just upgrade that file (and anything that depends on it) */
    done=_upgrade(path);
  } else {
    /* Auto upgrade everything that has changed */
    done=auto_upgrade(DRIVER);
    done+=auto_upgrade(AUTO);
  }
  /* This has to be done as a call_out, unfortunately.  The 'compile' kfun
     doesn't replace the objects with the new version until after the current
     thread has finished.  But we want to call upgraded() on the new version. */
  call_out("call_upgraded", 0, upgraded_clones+({ }));
  upgraded_clones=({ });
  return done;
}

void call_upgraded(string *paths) {
  /* Tell each object it has been upgraded.  Also move them to the void and back
     so uninit() and init() get called */
  int i;
  object obj;
  object old_location;

  for (i=0;i<sizeof(paths);i++) {
    obj=load(paths[i]);
    if (obj->is_clone() && obj->is_physical()) {
      old_location=obj->query_property(UNIV_PROP_PARENT);
      obj->move(nil);
    }
    obj->upgraded();
    if (obj->is_clone() && obj->is_physical()) {
      obj->move(old_location);
    }
  }
}
</pre>
<hr>
<pre>
Date: Wed, 22 Mar 2000 16:04:16 -0800 (PST)
From: Par Winzell
To: DGD Mailing List
Subject: Re: [DGD]Ojbect ID

Kevin N. Carpenter writes:

 > Back last June, Zell commented on using IDs to track objects using a two
 > dimensional model.  I presume he was referring to using a
 > ::status(obj)[O_INDEX] to retrieve the ID of the master object.  If I
 > understand the "master object" concept correctly, that would give me the
 > unique id of the master object something was cloned from.  Neat way to get
 > back to that object instead of manually striping the #xx off the cloned
 > objects name.

You're correct, but you miss the most important point. Upgrading works
by destructing the target program and recompiling all the leaf objects
that depend on it. Between the time that the program is destructed and
the last leaf is recompiled, there are two programs in memory, sharing
the same object name, one destructed and one not. The old program is
not destructed until no more clones/programs depend on it. If all the
leaf objects don't compile, both programs will hang around in memory
until you fix whatever errors are hindering the compilation. Quite in
general, you can have almost any number of old unfreed programs, each
haunting the system due to some lingering dependency.

The only way that old program is going to get freed is if you know
precisely what the dependency is, so you can clear it up. This in turn
requires a database of programs. Since clearly object name cannot be
used to index the program (all the unfreed issues have the same name)
the index becomes absolutely vital. It is much more than a clever
trick to get at the master object. :)

 > Unfortunately, I'm looking for a way to track clones when a
 > master object is upgraded.  The only way that was occurred to me is to build
 > a (doublely) linked list.  That's not a big deal, but if there is a cleaner
 > way that isn't limited by the config file array_size parameter, I'd like to
 > hear about it.

I treat clones as a completely separate matter. I do keep just such a
double linked list of clones, anchored in the clonable master object,
and I think others do as well. The kernel library maintains a link of
objects anchored in the owner, I believe. I don't see anything less
than neat about it...

 > Hmmm, one VERY crude method would be to search for all possible clones,
 > looping from #1 to ST_NOBJECTS doing find_object() to see if it exist.
 > Linked list would be a lot cleaner <grin>.  If I anchored the link list in
 > the master clone or master inheritable, that would make them need to be
 > recompiled rather than simply destructed, but I don't see that as a big deal
 > - is it?

Only programs are recompiled. If you have a thousand clones of foo.c
and you recompile foo.c, all the clones 'get' the new program. Apart
from debug/emergency purposes (which are good enough reasons in of
themselves), there are two occasions I can think of where you would be
happy to have such a linked list:

  A) you probably want to forbid the destruction of a clonable program
as long as there are existing clones of it -- otherwise you lose the
ability to recompile that program -- and that's testable as a pleasant
side-effect of maintaining that linked list

  B) when you need to change not just the program of a clonable but
also the data structure -- typically you write a patch() function in
the clonable that performs the data manipulation; upgrade the program
and then step through the list of all clones and call patch() in each
and every one.

Zell
</pre>
<hr>
<pre>
Date: Thu, 23 Mar 2000 12:03:30 +0100 (CET)
From: "E. Harte"
To: DGD Mailing List
Subject: Re: [DGD]Inherit_Program called twice?

On Wed, 22 Mar 2000, Kevin N. Carpenter wrote:

> I have a stripped down mudlib I'm using to develop my object manager and
> have just spent the last hour trying to figure something out.  It appears
> that the driver function "inherit_program" is being called twice during an
> object compilation.  I highly suspect this is something in my code, because
> that doesn't make any sense, but I sure can't find it.  Only 4 objects are
> involved: The driver, inheritable auto, object user, and inheritable by user
> userbase object.
> 
> The driver object forces a compile of the auto object.
> The driver object forces a compile of the user object.
> The user object inherits a file called userbase which forces a call to the
> driver function inherit_program().
> The inherit_program() forces a compile of the userbase object.
> That compile finished, along with some book keeping.
> ** Then, to my surprise, it appears that inherit_program() is called in the
> driver a second time for the same file, user, for the same program,
> userbase. **
> Since userbase is already compiled, this finishes quickly, but it still
> seems odd.
> I scanned all my code, it definitely DOES NOT call inherit_program directly.
> 
> Any ideas?  This is running under Redhat Linux 6.1 on an Dual Processor
> server.

Yes, it's the normal way of things to happen in DGD, nothing to do with
your OS or hardware. :-)

DGD only fully compiles one object at a time.

If you compile object A which inherits object B which has to be compiled
first, it'll finish compiling object B and then start the compilation of
object A from the start.

If you add debug statements in the path_include() driver-function you'll
find it calls those functions again as well.

Hope this helps,

Erwin.
-- 
Erwin Harte  -  Forever September, 2396 and counting  -  harte@xs4all.nl
</pre>
<hr>
<pre>
Date: Thu, 23 Mar 2000 13:36:27 +0100 (CET)
From: "Felix A. Croes"
To: DGD Mailing List
Subject: Re: [DGD]Tracking auto_object usage

"Kevin N. Carpenter" wrote:

> Since the driver function inherit_program() isn't called for the inheritance
> of the auto object, is there an alternative to having the object manager
> simply keep track of all compiled programs?  Obviously it could be made
> smart enough to not track other inheritable, since they would only get the
> auto object as part of the program that was inheriting them.

Sorry, but I don't understand how the part after the first comma
follows from the part before.

There is a very simple way to make sure the mudlib is properly notified
about the compilation of the auto object, without a need to have
inherit_program() called: mask the compile_object() kfun, and whenever
a non-auto object is compiled with the auto object destructed, perform
the same task as in inherit_program().


> Any opinions on the risk associated with keeping tabs on this via a mapping
> instead of a linked list?  The number of objects created via compiling
> rather than cloning is hopefully a small subset of the objects in the mud.
> Then again, I suppose its unwise to code in any limits around the config
> file parameter array_size if it can be avoided.

If you register objects in a single mapping, you cannot register more
objects than the maximum array/mapping size.  The decision is up to you.
DGD has a (configurable through recompiling) 64K limit on the number
of objects, and a (hard) limit of 32767 elements in an array or mapping.

Regards,
Dworkin
</pre>
<hr>
<pre>
From: Mikael Lind
Date: Thu, 23 Mar 2000 16:31:09 +0100 (MET)
To: DGD Mailing List
Subject: Re: [DGD]clones, master objects, inheritable, etc...

These are my corrections to Kevin's sanity check. I hope they're correct!

On Thu, 23 Mar 2000, Kevin N. Carpenter wrote:

> Master objects contain re-entrant, re-usable code and their own variable
> space.
>
> Clones contain a variable space and point to the master object code,
> saving the mud that code space.  Since the code space is shared, when
> the master object is changed, all clones see that change.

Correct. (As long as you don't destruct the master object and keep some
clones of it.)

> Inheritable objects are similar to master objects, except that when it is
> inherited, a copy of the inheritable object is included into the code space
> of the master object inheriting it.  Thus, if a master inheritable is
> changed, all objects previously inheriting it must be recompiled to force
> the changed code to be included.

Almost correct. However, the code isn't actually included. Instead, each
master object holds references to the proper versions of its inherited
programs. (Saving memory.) Old versions are thrown away when they are no
longer needed.

> Include files are just that, code included.  When changed, all master
> objects or inheritable that use the include file will need to be recompiled.

Correct.

> The driver code keeps track of some of this for you, calling recompile(obj)
> if obj is out of date with respect to something it inherits.  This call
> only occurs when another object that inherits obj is compiled.

Actually, I only think this will happen if there is a version clash
between several inherited objects. Example:

  A inherits nothing. (Okay, maybe the auto object.)
  B inherits the current version of A.
  C inherits an older version of A.
  D wants to inherit both B and C.

  When D is compiled, recompile(C) will be called in the driver.

> "nomask" functions cannot be overridden by any code inheriting them.
> 
> "atomic" functions will revert the state of the mud to its state prior
> to the function being executed if the function fails.
>
> "private" functions are local to an object and cannot be called via
> call_other(a,b) (or its syntactical equivalent: a->b).

Correct. (Local to a program, even. Private functions and variables can't
be accessed by inheriting objects.)

> "static" variables retain their last value between function calls to
> an object.

Not really. What you mean is global variables. The only effect of
declaring a (global) variable static is that it won't be saved or restored
by save_object() and restore_object(), respectively.

> Am I missing any other function or variable types (ignoring int, string,
> etc.)?  Have I goofed any of the object types?

I have no idea. :)

// Mikael Lind (Elemel)
</pre>
<hr>
<pre>
From: DGD Mailing List (Felix A. Croes)
Date: Mon Jan 14 23:05:01 2002
Subject: [DGD] New object issues

Noah Lee Gibbs wrote:

>   This is an objectd question, as you'll likely guess :-)
>
>   I'm working based on the Kernel MUDLib.  Am I guaranteed that when an
> object is recompiled, it will be recompiled with the same issue number
> (the number returned by status(obj)[O_INDEX]) as the previous version?

Yes -- both the object index and any object references (which use the
index in DGD internally) remain valid.

Regards,
Dworkin
</pre>
<hr>
<pre>
From: DGD Mailing List (Par Winzell)
Date: Tue Oct 14 08:06:00 2003
Subject: [DGD] Recompiling inherited objects

> My initial plan for solving the problem was this: For each object that is
> inherited I keep a list of the program-names (masters only) that inherit
> that object. I do this through the inherit_program() function in the
> driver object. When I compile an existing program that according to my
> data-mapping is inherited by other objects, I go ahead and outright
> destruct every master object with the names stored in my mapping. When
> done and compiled, I can compile all the objects I had to destruct in
> order to allow the desired recompilation.

The kernel library divides master objects into pure programs (those that 
are stored in lib subdirectories) and ones with potential dataspaces. 
One of the benefits of this separation is that pure programs can always 
be destructed without consequences because there is no data to lose.

Master objects with namespaces -- e.g. daemon objects -- you obviously 
don't want to have to destruct every time you upgrade a basic library 
object. These objects are, instead, recompiled in place, which also 
frees the reference to the old inheritable, replacing it with the new.

> This executes without errors, however, it does not have the desired
> effect. Do I understand it right if I think that the problem is that all
> the clones of the masters that inherit the code in question have their
> link to the original source code severed when their master is destructed?
> When I compile the source again, the clones of the old masters are not
> updated.

Yes, destructing a clonable while there are still clones left is a very 
bad idea. You should almost certainly forbid it. Again, the link from 
the master clonable to the inheritable you're upgrading can be freed by 
recompiling the clonable in-place.

> What I wonder then is: Is the solution now to also keep a list of all clones
> that inherit a certain  program, and when the new master has been
> compiled also recompile everyone of the old clones!? This seems a bit
> heavy to me and I am not sure it would work.

You can't recompile a clone and you don't need to. When you recompile 
the clonable, you immediately and automatically replace the master 
clonable for all those clones. You can have a mudlib with one clonable 
and a million clones of that; upgrading all clones is instantenous (at 
least from your point of view; technically they will be upgraded when 
they are swapped in).

A brief example:

   T: /lib/toolkit
inherited by
   D: /sys/daemon
and
   L: /base/lib/physical_object
which in turn is inherited by
   O: /base/obj/physical_object

So T's inheritance tree has three nodes: D, L and O. L is an internal 
node, and D and O are leaf nodes.

To recompile T, destruct all pure programs that descend inclusively from 
T, i.e. T and L. Then walk through the list of all remaining objects -- 
non-pure master objects and/or clonables -- and recompile them with a 
straight compile_object().

When you recompile the last object, the last reference to the old T 
disappears, the program is freed.

Tip: Put in debug messages in the callback functions that inform you 
that a program has been freed, so you make absolutely sure that the old 
version really is free. You might even want to enforce this in your code.

Tip: I assume you are already doing this, but remember that you can't 
index things just by program name. You can have 10 versions of 
/lib/toolkit in memory, all destructed, but 9 of them still unfreed 
because some object has yet to be recompiled. It is absolutely vital 
that you can get a list of unfreed-yet-destructed programs at any time.

The object index is available through status(obj)[ST_O_INDEX] or 
something similar to that.


There, that should keep you amused, and I am going to miss my bus, shit.

Zell
</pre>
<hr>
<pre>
From DGD Mailing List  Thu Apr  8 21:37:30 2004
From: DGD Mailing List (Mercy)
Date: Thu Apr  8 21:37:30 2004
Subject: [DGD] yet another object manager question.. or two

Hey guys,
I've spent the last three or four days thinking about object managers, 
and inheritance.
Basically I'm planning on writing a new object manager to support a 
persistant mudlib.
I've been reading over the archives and have a pretty good idea of what 
I need to do, and have been checking out Noah's objectd.c to see how he 
implements things.  So far so good, and I think I'm almost ready to 
start, but before I jump in the deep end, so to speak, I just want to 
see if I've missed anything obvious.


The object manager should know which inheritables a clonable inherits.
The object manager should know which inheritables are inherited by which 
inheritables.

Recompiling an inheritable should cause the object manager to:
destruct and compile the inheritable
recompile any cloneables (and LWO master objects) that use the inheritable
and..
repeat the process for each inheritable that inherits the newly 
recompiled inheritable.

Compiling and recompiling clonables and LWO master objects requires no 
special attention as clones and LWOs will automagically get updated.

The object manager should probably also keep track of clones and issues 
(... and LWOs? I dont think so..) depending on how I implement the 
above, right?


Are there any really obvious things I've missed out, or gaps in my 
logic?  Any hints would be greatly appreciated.  This is obviously just 
planning, I've yet to nail down a specific way of implementing 
everything.  I'm liking Noah's objectd as an example of using API_OBJREG 
for tracking things in a linked list.. I was starting to wonder about 
how I was going to implement that stuff.. and I see that alot of stuff 
is already handled in the kernal lib.  Excellent.

Since my name is Felix too, I'm just gonna use my nick to avoid confusion.
Thanks in advance,
Mercy
</pre>
<hr>
<pre>
From: DGD Mailing List (Noah Gibbs)
Date: Fri Apr  9 11:40:01 2004
Subject: [DGD] yet another object manager question.. or two

  Your object manager stuff sounds good, at least as a quick sketched
outline.  There are a few issues you haven't mentioned (no surprise,
there are a lot of random issues).  For instance, you'll need to decide
how to populate your data structures when the ObjectD comes on-line... 
Remember that several things, including the ObjectD, had to be compiled
before the ObjectD could run at all.  I believe Geir Harald Hansen has
his ObjectD hardcode some reasonable defaults and tells you to bring up
the ObjectD as quickly as it's possible to do so.  Mine recompiles
everything in an opening pass, though that still has at least one bug
in it as well, which I haven't tracked down -- each of those
inheritables winds up logging a message later when they're recompiled. 
Not sure why, and it doesn't seem to cause any actual harm.  Anyway.

  And yeah, you're right -- the kernel lib will supply some very useful
information for you while you're doing this stuff.  It's a good place
to look for lists of objects.
</pre>
<hr>
<pre>
From: DGD Mailing List (Noah Gibbs)
Date: Fri Apr  9 13:46:00 2004
Subject: [DGD] yet another object manager question.. or two

--- Michael McKiel wrote:
> I believe hardcoded defaults, like hardcoding kernel dependencies et
> al was
> mentioned in the archives, is there any reasonable reason not to do
> this?
> these are things that should likely never change.

  There are two reasons.  One is that the kernel library might change
in some way, which would change the correctness of the defaults.  The
other is that the code might change (for instance, another object might
be required by the ObjectD or wanted first and compiled before it), and
if somebody knowledgeable about the ObjectD doesn't change that, you'll
get some weird unexplained bugs.

  I don't like non-local effects (where you change something and
something apparently-unrelated elsewhere changes).  The problem with
hardcoded defaults is that you get non-local bugs if they're not
perfect.

  My way *does* require compiling about seven files twice.  However,
Phantasmal is so big at this point that the amount of double-compiled
code isn't a significant part of the whole.
</pre>
<hr>
<pre>
From: DGD Mailing List (Noah Gibbs)
Date: Fri Apr  9 13:49:00 2004
Subject: [DGD] yet another object manager question.. or two

--- Michael McKiel wrote:
> [...] there was a
> fairly recent discussion of mapped arrays and using 
>     ([ clone_num / 1024 : ({ objects }) )]; 
> Linked list vs array/array mapped/array seems to be more of a
> personal preference of which you prefer to deal with.
> Mapped Arrays to me is the cleanest/easiest approach to many
> things requiring data storage.

  My approach to this was to make a DGD object which I called a 'heavy
array' that does something similar.  So rather than having a hardcoded
array of LWOs for tracking data, I have a collection object I add the
LWOs to.  Never did anything really useful with it (never needed to,
didn't have that many objects), but it would be very easy to make it
implement mapped arrays, as you have here.
</pre>
<hr>
<pre>
From: DGD Mailing List (Stephen Schmidt)
Date: Fri Apr  9 14:02:01 2004
Subject: [DGD] yet another object manager question.. or two

On Fri, 9 Apr 2004, Michael McKiel wrote:
> I believe hardcoded defaults, like hardcoding kernel dependencies et al was
> mentioned in the archives, is there any reasonable reason not to do this?
> these are things that should likely never change.

If you're planning to distribute your code to others, then it's
very hard to know what some idiot who downloads your lib might
change :)

Hardcoding things makes them brittle. Brittle is not necessarily
bad, as long as you can be certain changes will never be made
that break the brittleness. As long as you are the only user
of your code, you can be reasonably sure about that. Or at worst,
you can hope that you'll remember to change the hard coding,
perhaps only after the first crash.

However, if you are going to let someone else use your code,
either because you're distributing it, or because someday your
interests are going to change and someone else is going to take
over as head wizard at the mud - then making things robust can
make life easier for the person who comes after you. Or, it can
make life easier on you if, like me, you tend to forget what
you hardcoded in the past. That's the main reason to not hardcode
things. It also makes it easier for you to go back and do changes
at fundamental levels without having to rewrite all your hardcodes.

Steve
</pre>
<hr>
<pre>
From: DGD Mailing List (Erwin Harte)
Date: Fri Apr  9 17:34:01 2004
Subject: [DGD] Re: yet another object manager question.. or two

On Fri, Apr 09, 2004 at 05:56:10PM -0400, Michael McKiel wrote:
[...]
> I think it would be fairly easy to 'hardcode' the defaults safely, if such
> was needed, but there are no dependancies, the files compiled pre-objectd
> are:
> FILE     Dependancy/Inherits
> auto     none
> driver   none
> objregd  none
> rsrcd    none
> initd    none
> objectd  none

You seem to be overlooking the indirect dependencies.  This is the
list I use for my object-db:

    DRIVER, AUTO,

    OBJREGD, RSRCD, RSRCOBJ, ACCESSD, USERD, SYS_INITD, LIB_USER,
    API_ACCESS, API_USER, API_RSRC, API_OBJREG, SYS_OBJECT, LIB_CONN,

    LIB_WIZTOOL,

    DEFAULT_WIZTOOL,

    TELNET_CONN, BINARY_CONN,

    DEFAULT_USER,

    SYS_OBJECTD

All #defines taken from the relevant /include/kernel/*.h files.

And this is the inherit information I'd use if I ever have to
cold-start again:

    private string *
    preloaded_inherits(string str)
    {
    switch (str) {
    case DRIVER:
    case AUTO:
        return ({ });
    case OBJREGD:
    case RSRCD:
    case RSRCOBJ:
    case ACCESSD:
    case USERD:
    case SYS_INITD:
    case LIB_USER:
    case API_ACCESS:
    case API_USER:
    case API_RSRC:
    case API_OBJREG:
    case SYS_OBJECT:
    case LIB_CONN:
        return ({ AUTO });
    case LIB_WIZTOOL:
        return ({ API_ACCESS, API_USER, API_RSRC });
    case DEFAULT_WIZTOOL:
        return ({ LIB_WIZTOOL });
    case TELNET_CONN:
    case BINARY_CONN:
        return ({ LIB_CONN });
    case DEFAULT_USER:
        return ({ API_ACCESS, API_USER, LIB_USER });
    case SYS_OBJECTD:
        return ({ API_RSRC, API_OBJREG });
    default:
        error("Internal error, missing preloaded_inherits() entry");
    }
    }

It's not difficult, but you don't want to miss any files, no. :-)

Cheers,

Erwin.
-- 
Erwin Harte
</pre>
<hr>
<pre>
From: DGD Mailing List (Noah Gibbs)
Date: Thu Apr 15 23:34:08 2004
Subject: [DGD] objectD differences...

--- Mercy wrote:
> I figure my options are more or less to use Noah Gibbs' or Geir
> Hansen's objectDs.

  Yeah, basically.  Somebody else did one at one point (John McKenna,
perhaps?), but it was never seriously tested.

> Noah's would require a bit of effort to get working outside
> of Phantasmal, from what I can tell, but seems to be more
> featureful.

  Yes and yes.  Though getting it working outside of Phantasmal won't
be too bad.  You need to take its tracking classes with it, the LWO
types for issues.  But that's not really a Phantasmal thing -- they're
used only by the object manager, they're just not in objectd.c.  The
only other issue that comes to mind is that it uses the Phantasmal
LogD.  The amount of that that you'll need is pretty small, but you
could always tear that out and replace it with your own logging stuff.

> Would I lose much by simply using Geir Hansen's object
> manager, given 
> that for the time being, full persistance really isnt a
> major concern for me?

  The only significant feature you're losing is the "upgrade" and
"upgraded" functions, which are really only needed for full
persistence.

> Are there any traps I should watch out for in using someone
> else's object manager?

  Just that nobody's object manager is fully tested.  Phantasmal's has
a couple of pretty benign known bugs (I already mentioned them on this
list).  I don't know of anybody having tested Geir Hansen's as
seriously, but it presumably works about as well -- Phantasmal's hasn't
actually had a bug in a long time that wasn't related to new
functionality.

  The other thing to consider, come to think of it, is that
Phantasmal's ObjectD has more logging and will detect more weird
anomalous situations for you, at least if you check your logs.  That's
only a problem if something goes wrong, though :-)

> I think getting 
> some code, other than an object manager, written, would help
> me test any 
> object manager I write for myself anyway,

  Absolutely.  Good plan.
</pre>
<hr>
<pre>
From: DGD Mailing List (Mercy)
Date: Thu Apr 15 23:48:01 2004
Subject: [DGD] objectD differences...

Noah Gibbs wrote:

>--- wrote:
>  
>
>>I figure my options are more or less to use Noah Gibbs' or Geir
>>Hansen's objectDs.
>>    
>>
>
>  Yeah, basically.  Somebody else did one at one point (John McKenna,
>perhaps?), but it was never seriously tested.
>  
>
Pete-someone, I believe, and I just downloaded his code from your site, 
along with BBLib and some other stuff I'd been overlooking all the while.

>>Noah's would require a bit of effort to get working outside
>>of Phantasmal, from what I can tell, but seems to be more
>>featureful.
>>    
>>
>
>  Yes and yes.  Though getting it working outside of Phantasmal won't
>be too bad.  You need to take its tracking classes with it, the LWO
>types for issues.  But that's not really a Phantasmal thing -- they're
>used only by the object manager, they're just not in objectd.c.  The
>only other issue that comes to mind is that it uses the Phantasmal
>LogD.  The amount of that that you'll need is pretty small, but you
>could always tear that out and replace it with your own logging stuff.
>  
>
That would probably be a good learning process, I mean, obviously 
changing some calls to the logger isnt going to be very taxing, but its 
a start, and as you point out further down, your manager's logging would 
come in handy at some point, more than likely.

>>Are there any traps I should watch out for in using someone
>>else's object manager?
>>    
>>
>
>  Just that nobody's object manager is fully tested.  Phantasmal's has
>a couple of pretty benign known bugs (I already mentioned them on this
>list).  I don't know of anybody having tested Geir Hansen's as
>seriously, but it presumably works about as well -- Phantasmal's hasn't
>actually had a bug in a long time that wasn't related to new
>functionality.
>
>  The other thing to consider, come to think of it, is that
>Phantasmal's ObjectD has more logging and will detect more weird
>anomalous situations for you, at least if you check your logs.  That's
>only a problem if something goes wrong, though :-)
>  
>
If, or when? ;)

Thanks for the speedy reply, it occured to me that I was/am kind of 
jumping the gun on the ObjectD thing... I dont really need one while I'm 
constantly starting and stopping the driver and have very little code 
written.. the Kernel lib would probably handle most of what I need at 
this very early stage.  That said, it wouldnt hurt to have a working 
object manager plugged in, for when things get more complex, I'm sure.

Really I'm still trying to work out how I'm going to structure the lib, 
within the restrictions of the Kernel Lib, and I'll probably end up 
starting over once I get a proper feel for how everything works.  It's 
all too abstract until I start throwing code together.  :)

Mercy
</pre>
<hr>
<pre>
From: DGD Mailing List (John West McKenna)
Date: Fri Apr 16 04:37:01 2004
Subject: [DGD] objectD differences...

Noah Gibbs writes:

>  Yeah, basically.  Somebody else did one at one point (John McKenna,
>perhaps?), but it was never seriously tested.

It had a nasty bug that I never tracked down, but more fatally it didn't
use the kernel library.  Just pretend it doesn't exist.

John
</pre>
<hr>
<pre>
From: DGD Mailing List (Steve Wooster)
Date: Fri Mar 26 19:17:01 2004
Subject: [DGD] Design question about inheritance code

     I'm trying to think about how I want to code a daemon to keep tack of 
inheritance trees, and there are a couple of ideas I came up with, for 
which I'm trying to think of good/bad things about them...

1. I'm thinking of making it so that a file can't be removed while an 
object is loaded from it. If an object is loaded from it, you need to 
destruct the object before you can remove the file. I figured this might be 
good for persistent muds, because it makes it impossible for a forgotten 
object, "/path/dir/xyz.c" to be floating around, using up memory if it 
doesn't exist in the file-system (I assume my mudlib is the only thing with 
access to the mud's directory).

2. This idea is a bit more iffy... perhaps when an object/lib/etc is 
compiled, a copy of the source code is saved in memory somewhere. When a 
full update occurs (like if auto.c is being recompiled or something), the 
object is recompiled from the source code in memory rather than the file. I 
figured this might be good, because if you're messing with a currently 
loaded file, and unbeknownst to you, somebody updates auto.c or a lib file, 
the object will be recompiled with the version of the code loaded up in 
memory rather than the new experimental version on disk. On the other hand, 
I sort of think it might not really be necessary, and could use up a lot of 
extra space (I figure I'd keep the source-file info in lightweight objects, 
so I would only have the necessary source codes loaded up in memory for any 
given thread).

Any comments or other ideas? Thanks.

-Steve Wooster 
</pre>
<hr>
<pre>
From: DGD Mailing List (Felix A. Croes)
Date: Sat Mar 27 05:35:01 2004
Subject: [DGD] Design question about inheritance code

Steve Wooster wrote:

>[...]
> 2. This idea is a bit more iffy... perhaps when an object/lib/etc is 
> compiled, a copy of the source code is saved in memory somewhere. When a 
> full update occurs (like if auto.c is being recompiled or something), the 
> object is recompiled from the source code in memory rather than the file. I 
> figured this might be good, because if you're messing with a currently 
> loaded file, and unbeknownst to you, somebody updates auto.c or a lib file, 
> the object will be recompiled with the version of the code loaded up in 
> memory rather than the new experimental version on disk. On the other hand, 
> I sort of think it might not really be necessary, and could use up a lot of 
> extra space (I figure I'd keep the source-file info in lightweight objects, 
> so I would only have the necessary source codes loaded up in memory for any 
> given thread).

My mudlib does this.  It has two types of upgrades, one to upgrade from
the current source file, and one to upgrade using the source code that
the object was compiled from.  For example, suppose I am working on an
object B, editing the source code, and someone else upgrades object A
and everything that inherits from it, which includes object B.  If B
were to be recompiled from source code it might not even compile at
all.  In this case, B ought to be recompiled from the same source code
that it was originally compiled from, without disturbing my own work
on the current source code.

Another situation: suppose that different issues of B exist, compiled
from different versions of the source code.  Then I'd like to be able
to retrieve each of those versions to look at the differences between
them.

Regards,
Dworkin
</pre>
<hr>
<pre>
From: Par Winzell
Date: Sat, 19 May 2001 08:06:46 -0700
Subject: Re: [DGD]inherit_list

 > The other thing I could do is hook into inherit_program in the driver
 > and keep my own inherit list, but that seems like a bit of a mess.

That is precisely the thing to do. It's not a "mess" at all, it is
a very elegant solution, but it -is- a bit of an undertaking to get
all the kinks worked out.

 > I see in an old mail from Dworkin that he recommended keeping an
 > inherit list database in the mudlib.
 > Is that still recommended?

Yes.


 > I would expect that DGD knows the inherit chain already - wouldn't it
 > be more efficient to be able to ask the driver, rather than track it
 > yourself?

Probably, but DGD is still a minimalist design. If it can be done
quickly and well in LPC, why clutter up the driver?


 > And on a separate issue, why does the kernel lib require that you have
 > write access to an object to compile it? 
 > It would appear to be a read operation? Why write? 

It "reads" the source file, but it certainly modifies the object. :-)

Zell
</pre>
