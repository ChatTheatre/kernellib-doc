---
title: Directory Structure
layout: default
---

Where do I <b>put</b> all this stuff?

You've probably asked yourself that several times already, just
getting started. When you make a new library, or a new clonable, or
a new data file, where do you <i>put</i> it? This document can get
you started.

Where you put stuff is surprisingly important. The Kernel MUDLib
hardcodes a <i>lot</i> of behavior according to the path to a given
file. That includes a lot of information about who has access to
what, and whether an object is inheritable, clonable or an LWO. So
let's hit some basic guidelines.

## Where do specific object types go?

You'll need to put a "/lib/" directory somewhere in the path to
every library (inheritable). The Kernel MUDLib requires it. It's
good practice. It's good documentation. Do it. You'll also need to
<i>not</i> put "/obj/" or "/data/" anywhere in that path. For more
information about those paths, and whether an object is cloneable
or inheritable, see the page about <a href=
"Inheritance.html">Inheritance</a> in the Kernel Library. It'll
explain what's mandatory.

Daemons are objects like any others, but they're meant to be
used by doing a find_object on them and calling their methods with
call_other. Daemons often wind up in a directory with "/sys/" in
the name instead of "/lib/", "/data/" or "/obj/". This is purely
optional, mostly a form of documentation. You're just saying that
you plan to use objects in that directory as daemons. It's still a
good idea.

## What about Data?

This one's important too, but for different reasons. You don't
need to have a particular piece of your path to open a file with
read_file(). But different paths require different permissions to
access. For instance, stuff in "/usr/System" is readable and
writable only by the privileged admin user (by default) and by
anybody you explicitly grant access to. In general, permissions are
given per-directory You'll want to set aside a place somewhere
under /usr/ to put any piece of data, because generally only code
in that same user's directory can read it (or everybody can, if you
put it in "/usr/Foo/open/").

Files with passwords in them are a good example. While passwords
are encrypted, they're not unbreakable, so often you want to avoid
giving people read access. That keeps them from using a password
cracking program, or at least makes it harder. You may need to give
certain wizards read access to the System directory but not want
them to be able to read all the save files. By putting the save
files somewhere else (say under /usr/Secret or something), you can
prevent anybody that doesn't have full admin permission from being
able to read or write the user data files. Note that you'll have to
write your password code carefully so that <i>it</i> can still read
it!

## What directories does the Kernel Library treat specially?

The Kernel Library checks various things about the path in many
of its operations. Let's look at different operations, and what the
Kernel Library checks about the path for each one.

<ul>
  <li>A lot of Kernel MUDLib functions can only be called from
  objects under /usr/System. Objects under /usr/System are also
  allowed to read and write a lot of things automatically, without
  having to check permissions. They're also the only objects that
  can inherit from objects under /kernel. Only objects under
  /usr/System may destruct objects that they don't own, though even
  they can't destruct an object under /kernel. Only objects under
  /usr/System are allowed to call swapout(), dump_state(), and
  shutdown() usefully.</li>

  <li>Only objects <i>outside</i> the /usr/System directory can
  have a special nonstandard AUTO object.</li>

  <li>The /kernel and /include/kernel directories and their
  subdirectories are guaranteed writable to nobody. Files under
  /kernel or /include/kernel may not be written to or deleted.
  Directories cannot be made or destroyed there, and files cannot
  be renamed to or from those directories. Objects under /kernel
  can only be cloned by other objects under /kernel.</li>

  <li>You can't call_other on any object with /lib in its path. You
  can't find those objects with find_object(), you can't call their
  functions. You can only inherit objects with /lib in their paths
  and without /obj/ or /data/ in their paths.</li>

  <li>Files with /include in the path can always be included,
  regardless of read access. (TEST ME, with and without a '..' in
  the path)</li>

  <li>Files outside of /usr/System are allowed to have a special
  nonstandard AUTO object. If a non-System file includes AUTO from
  /include/std.h, and an objectd is registered, the Kernel Library
  will call the path_special() function of the objectd to determine
  what AUTO object should be used. If a value other than "" or nil
  is returned, it is assumed to be the path to the nonstandard AUTO
  object. This can be used for scripts or other controlled access
  since built-in functions can be overridden by an AUTO
  object.</li>

  <li>Objects in /kernel have no maximum rlimits(), so they could
  theoretically loop infinitely or crash for lack of stack space.
  In practice, this essentially never happens. Objects in
  /usr/System may voluntarily choose not to limit their number of
  ticks in rlimits(). This means they may occasionally loop
  infinitely, especially if coded poorly.</li>

  <li>Files outside /usr, or in /usr/foo/open (for any value of
  foo) are readable to everybody. Except stuff under /kernel/,
  that's not necessarily open to reading.</li>

  <li>A user has full access of every object and file under his own
  home directory. Objects under that user's directory have full
  access to each other, and to files under the user's home
  directory.</li>

  <li>The create() function that gets called automatically on an
  object will be called with the 'clone' argument only if /obj/ or
  /data/ is in the object's path. Otherwise it will be called with
  no argument.</li>

  <li>Libraries not under /kernel require you to have read access
  to compile them. Non-libraries, anything under /kernel, and
  anything compiled from sources requires write access instead of
  read access if you want to compile it. There's an exception if
  the object doing the compiling is under /usr/System. And anything
  not under a /usr/XXX directory can be compiled by anybody.</li>

  <li>An object must have an /obj/ in its path, but no /data/ or
  /lib/, to be clonable. Objects outside of the /usr/XXX/ area
  can't clone another object. Objects that aren't under /usr/System
  need read access to an object to clone it.</li>

  <li>Only objects in /usr/System can pass a uid argument to
  clone_object or new_object. That's a way to change who owns the
  new clone or LWO. You should probably never do this.</li>

  <li>An object to be cloned as an LWO with new_object must have
  /data/ in the path, but not /obj/ or /lib/.</li>

  <li>Objects outside of /usr/System get an edited call_trace, one
  with no arguments included.</li>

  <li>Objects outside of /usr/System don't get arguments in the
  output of status(), and nobody is allowed to see the callouts of
  objects in /kernel.</li>

  <li>Certain Kernel objects get direct call_outs rather than the
  usual tracked ones. If you don't know what this means, don't
  worry about it. You'll sleep better.</li>
</ul>
<hr>

## Specific Projects

This is part of the Adding Stuff section, so what kind of stuff
should you add? One simple answer is "everything", since this
document tells you where everything goes. But there are also some
projects you can undertake to improve your MUDLib.

For instance, a simple thing you can do is to move your user
data from its current save location to another, as we suggest
above. Mainly you'll need to modify "user.c". Go give it a try!
<hr>
<pre>
From: dgd at list.imaginary.com (dgd@list.imaginary.com)
Date: Fri Oct 17 11:45:01 2003
Subject: [DGD] Methodology: Directory structure & Areas

>From what I've been able to gather thus far the /kernel directory is not to be 
modified unless you really know what you're about to do since it can very well 
take the entire MUD with it and possibly damage your security. This makes sense.

The /usr directory holds most of your code. The System directory holds trusted 
code and should only be modified by those who know what they're doing and can 
keep the code tight and secure. This also makes sense.

For some things, though, you want directories outside of the initial structure. 
For instance I have a /log directory for all of my log files, and a /save 
directory to keep player data (in a set of directories...one directory per 
player, and each file therein for each character they have).

What I'm debating with myself right now is where to put my domain directories. 
Older MUDs that I've seen just lets each builder keep their objects under their 
own directory trees. This is all well and good, but it encourages tinkering and 
can lead to errors. This is especially bad in terms of guild objects or 
commonly used items and areas. This is especially bad if your characters' 
equipment is persistent at all. That sword they carry may change from day to 
day.

Because of this you can come up with a domain structure. Thus each major area 
is run by a domain admin, and he/she moderates what code becomes real in the 
domain. When a builder wants to change stuff they do it and test in in their 
own directories, and then the changed objects are implimented by the domain 
admin (or maybe the head admin depending on how you setup your overall 
hierarchy).

My main question is where is the best place to put these? I can think of three 
general methods:

1) An external directory. Make a directory such as /domains and put each domain 
tree under that. Perhaps even have a different root directory for each domain, 
though that may get cluttered.

2) In the domain admin's directory. Thus if "bob" is in charge of the "hell" 
domain we'd have /usr/bob/hell as the root for that domain's stuff.

3) In it's own user directory. Thus you would have /usr/hell to hold the domain 
files for hell, and bob would have write access on it given the previous 
example. In this case, though, hell still wouldn't be an interactive user like 
System.

Which system do you guys think works best for this? My biggest question on #1 
is whether or not the kernel mudlib has any problems executing code that 
doesn't reside in /kernel or /usr, and if so is there an easy way to work 
around that?

BTW...this list rocks!

Well...back to making sure I have DGD's inheritence system down before I code 
much more. :)

Patrick
</pre>
<hr>
<pre>
From: dgd at list.imaginary.com (Stephen Schmidt)
Date: Fri Oct 17 11:59:01 2003
Subject: [DGD] Methodology: Directory structure & Areas

On Fri, 17 Oct 2003 Nihilxaos wrote:
> 1) An external directory. Make a directory such as /domains and put each domain
> tree under that. Perhaps even have a different root directory for each domain,
> though that may get cluttered.

This is wisest if the number of domains will be limited. If it
won't, then /domains/h/hell may be the best solution.

> 2) In the domain admin's directory. Thus if "bob" is in charge of the "hell"
> domain we'd have /usr/bob/hell as the root for that domain's stuff.

This is a poor idea, because when Bob resigns, or leaves to start
his own mud, or is caught handing out cash and power swords to the
newbie users, or comes out on the short end of an administration
power struggle, or is gone for whatever reason, the system collapses.
It also encourages Bob to think the domain is "his" and that will
lead to more administration power struggles in which Bob might be
forced out. Avoid this solution strenuously.

It's also worth noting that code ownership can be determined by
directory structure. A not-uncommon method of determining code
ownership is that Bob owns any files in /usr/bob and the mud
admin owns any code anywhere else. If so, then any code the
mud needs to keep if Bob leaves (like an entire domain ;) needs
to be outside /usr/bob. Otherwise, if you use the ownership
method suggested above, Bob can take his domain with him when
he leaves. This is unwise.

Of course, you can use other methods of arranging code ownership,
but you must do something to deliniate it, and the method of "your
directory is yours, everything else is the mud's" is pretty simple
and intuitive.

> 3) In it's own user directory. Thus you would have /usr/hell to hold the domain
> files for hell, and bob would have write access on it given the previous
> example. In this case, though, hell still wouldn't be an interactive user like
> System.

This is problematic if someone creates a character with the same
name as a domain, and makes wizard. Probably better is to use
/usr/hell for the wizard "hell" and /usr/Hell for the domain "hell".
That might be confusing but at least it's safe.

However, treating domains as if they were users is probably asking
for trouble in numerous ways. If you want to go down this road
you are almost surely better off using /domains/hell rather than
/usr/hell or /usr/Hell.

That said, I have a strong memory that the kernel lib makes
certain assumptions about objects in /usr (my memory does not
provide details, alas) and if you are working over the kernel
lib, then you have to satisfy its assumptions. At least, I
remember being told that when I asked more or less the same
question in 1998. Things may have changed since then. Experten,
can you fill in some details here?

Steve
</pre>
<hr>
<pre>
From: dgd at list.imaginary.com (Noah Gibbs)
Date: Fri Oct 17 12:52:01 2003
Subject: [DGD] Methodology: Directory structure & Areas

--- Nihilxaos wrote:
> /kernel directory is not to be modified

  Correct.

> The /usr directory holds most of your code.

  That's the common setup, though it's not mandatory. 
Basically the Kernel Library gives a bunch of
privileges and hooks to stuff in /usr/System, which
then farms it out any way it feels like.

  But in the usual setup, most of your code winds up
in /usr/System.  Phantasmal adds another layer called
/usr/common.  Note that due to the name, Phantasmal
also has to be careful to never let anybody make a
character, especially a wizard, named "common" :-)

> For some things, though, you want directories
> outside of the initial structure.

  Yup.  I'm a big fan of these, though the Kernel
Library seems not to be.  It's a matter of taste.  You
can get away with having none of them at all, or you
can use them for almost everything.

> What I'm debating with myself right now is where to
> put my domain directories. 
> Older MUDs that I've seen just lets each builder
> keep their objects under their 
> own directory trees. This is all well and good, but
> it encourages tinkering and 
> can lead to errors.

  Bear in mind that admins *should* be able to do some
amount of tinkering in their own areas, and their own
directories are one fine place to put that code.  With
that said, Phantasmal tries to encourage admins to
never put anything in their own directories, and
instead build directly in the MUD domain files.  If
you want a tinkering area, simply cut it off from
non-admin access by making sure there are no exits
that lead to it (and eventually through a "locked to
players" zone flag to avoid accidental teleportation).

> This is especially bad in terms
> of guild objects or 
> commonly used items and areas. This is especially
> bad if your characters' 
> equipment is persistent at all. That sword they
> carry may change from day to day.

  To some extent this is likely to happen anyway.  The
objects have to exist *somewhere*, and they almost
certainly need to be mutable.  But you're right,
putting the object into the home directory of the
creating admin is just asking for trouble.

> My main question is where is the best place to put
> these? 
> 1) An external directory.

  Obviously (if you look at Phantasmal) I'm in favor
of this.  I think it solves the problems best.  It's
also the least like the traditional LPMUD solution,
but I didn't come from an LPMUD background so I
couldn't care less.  I'm a big fan of the idea that
everything that anybody builds should be reasonably
viewable, even if it's not done yet.  I'm also a fan
of the idea that content and access privileges should
be decoupled -- it should be easy to grant and remove
access to things without having to create, destroy or
move them.

> Make a directory such as
> /domains and put each domain 
> tree under that. Perhaps even have a different root
> directory for each domain, 
> though that may get cluttered.

  Yeah.  I put all this stuff under a subdirectory and
then separate it out into zones (my equivalent of
domains) when saving.  So each domain is in a
subdirectory of the main one.  Then again, Phantasmal
has a rather odd setup for an LPMUD, and the code
doesn't wind up going in those directories (mostly).

> 2) In the domain admin's directory. Thus if "bob" is
> in charge of the "hell" 
> domain we'd have /usr/bob/hell as the root for that
> domain's stuff.

  I agree with Stephen Schmidt about why this is a bad
idea.  It's fine for only a single user, but in that
case why do you care where it goes?

> 3) In it's own user directory. Thus you would have
> /usr/hell to hold the domain 
> files for hell, and bob would have write access on
> it given the previous 
> example. In this case, though, hell still wouldn't
> be an interactive user like 
> System.

  Again, agree with Stephen Schmidt.  Easier not to
confuse your namespace of users.

> Which system do you guys think works best for this?
> My biggest question on #1 
> is whether or not the kernel mudlib has any problems
> executing code that 
> doesn't reside in /kernel or /usr, and if so is
> there an easy way to work 
> around that?

  The Kernel Library has no trouble executing code
anywhere.  It makes some assumptions about permissions
based on filenames, though -- something whose path
contains "/lib/", for instance, is guaranteed to be an
inheritable, not a clonable.  But everything about
reading, writing and executing can be set with the
various permissions commands, so if you want a
directory protected differently, just change its
permissions.

  At some point I have to get around to writing up the
Kernel Library's rules about pathnames and what they
mean...  I keep meaning to.  There's no serious
documentation on it that I know of, I just figured it
out by reading through the code.

> BTW...this list rocks!

  Thanks :-)  You're asking unusually good and
detailed questions, which definitely helps.  And you
seem to have read most of the obvious documentation,
so you're also making more progress in between
questions than the average petitioner.

  There's also the fact that the documentation's
better than it was.  I take significant pride in my
tutorial on building a MUDLib based on the Kernel, for
instance.

> Well...back to making sure I have DGD's inheritence
> system down before I code 
> much more. :)

  Yeah.  Really understanding it is tough, and pretty
much requires writing or debugging an Object Manager. 
Luckily, you've got at least two full-on Object
Managers to look at (mine and Geir Harald Hansen's)
for details, though they're both pretty obscure to
read through.

  Still, you can easily get the basics by reading the
Kernel Library stuff, or something based on it (like
Phantasmal).
</pre>
<hr>
<pre>
From: dgd at list.imaginary.com (Par Winzell)
Date: Fri Oct 17 13:26:01 2003
Subject: [DGD] Methodology: Directory structure & Areas

> 1) An external directory. Make a directory such as /domains and put each domain 
> tree under that. Perhaps even have a different root directory for each domain, 
> though that may get cluttered.

Keep in mind that if you use the kernel library, objects that are not in
/usr/* have very limited abilities. Somebody else should expand on this,
but over time it has become my strong impression that code that wants to
take an active part in the game should reside in a /usr directory.

> 3) In it's own user directory. Thus you would have /usr/hell to hold the domain 
> files for hell, and bob would have write access on it given the previous 
> example. In this case, though, hell still wouldn't be an interactive user like 
> System.

This would definitely be my suggestion.

Zell
</pre>
<hr>
<pre>
From: dgd at list.imaginary.com (Noah Gibbs)
Date: Fri Oct 17 16:05:01 2003
Subject: [DGD] Methodology: Directory structure & Areas

--- Stephen Schmidt wrote:
> The former seems to me to impose a large and
> unreasonable burden
> on file organization. There are many different types
> of files which,
> for one reason or another, need to write a file
> occasionally, and
> it seems forced to require that they all be in
> /usr/System, when
> they might well be logically grouped elsewhere.

  True.  Your file-access daemon is almost exactly the
approach I mean.  I don't temporarily grant access to
anything.  Rather, I have the ability to call a
function like save_user_file(), which will check who
calls it (with previous_program() usually) and either
do the appropriate thing or cause an error.  That
imposes an ugly burden on /usr/System since it needs
to know who will call what function, so I have an
additional layer called /usr/common which translates
fairly game-specific requests into more system-level
requests.  So:  /kernel defines the core OS-type
operations, /usr/System defines my limits on saving
and loading files, /usr/common defines the overall
rules of the game, and anything outside of that is
game-specific and has no actual privileges.  You could
add another level easily enough:  /usr/common could
define the kind of game and /usr/Innsmouth or
/usr/EyeOfTheDragon or /usr/BoboMUD or whatever could
define things specific to the game itself.

  The main problem with this organization, regardless
of how many or how few levels you use, is that you
have to know what function names will call you.  Which
means that each layer needs to know something specific
to the layer above it.  That doesn't make me happy,
but I don't currently have any more general
access-control method that I like.  I could make every
resource type be a user in the Kernel Library /usr
heirarchy, but that gets pretty ugly pretty quickly.

> Or, from what Noah wrote:
> 
> Noah Gibbs wrote:
> > > I get around it by
> > > having access-controlled functions in
> /usr/System that
> > > do stuff that requires more privilege.  Since
> code
> > > outside of /usr/ has no user directly associated
> with
> > > it, you need to either have it use only things
> with no
> > > permission controls, or have functions somewhere
> under
> > > /usr/ that recognize that code specifically.
> 
> I gather one can write an access manager that would
> temporarily
> grant permission to /cmds/wizard/delete.c to do the
> file access?

  In concept.  In actuality, you'd have a line in
FILEDAEMON:delete_file(), right at the beginning, that
checked previous_program() and determined whether to
allow the deletion.  It might recognize specific
files, or specific path prefixes.  This is essentially
the same way that the Kernel Library does it, but
without the abstraction of users and home directories,
and without some of the automation.
</pre>
<hr>
<pre>
From: dgd at list.imaginary.com (Noah Gibbs)
Date: Fri Oct 17 16:11:01 2003
Subject: [DGD] Methodology: Directory structure & Areas

--- Nihilxaos wrote:
> Well...to my understanding it's good to have
> anything that requires any sort of 
> trusted functionality reside in /usr/System anyway.

  Yup.

> The one thing this does 
> bring up, though, is that guild objects can either
> modify a character's save 
> file or have their own save file for the character
> (a thing I'm a fan of...more 
> damage localization in case of error).

  I make sure that all of this resides under
/usr/System at some level.  For instance, commands can
modify the user object, which will change the save
file next time something causes it to save.  As a
result, many functions that modify saved information
cause the user object to save as well.

  In concept, this stuff is all carefully protected. 
In practice, there are still some functions (like
set_locale()) that I need to protect properly.

  If you wanted guild objects to have their own save
files, they'd need to either call to something in
/usr/System (or other /usr/XXXX directory) that could
do the write for them, or the guild objects would have
to go into a /usr/XXXX directory.

  If I were going to add guilds to Phantasmal, I'd
probably add a /usr/common/guild.c object and have a
list of allowable guilds in an UNQ file somewhere.  If
an allowed guild object (like, one in the UNQ file)
registered itself and asked to load or save, then
/usr/common/guild would cheerfully do that operation. 
That means you've got access control (the UNQ file)
and the operations can occur outside /usr/System
(which is good when possible).  I'm not sure whether
I'd make a /usr/common/obj/guild object clonable and
instantiate guild objects for each guild, or whether
I'd have a /usr/common/sys/guildd that fielded
requests for the other guild objects.  There are some
kind of weird path restrictions on inheritance, so
it'd be tough to have objects outside the /usr
heirarchy inherit from /usr/common/lib/guild. 
Possible, but probably not worth the trouble.
</pre>
<hr>
<pre>
From: dgd at list.imaginary.com (Michael McKiel)
Date: Thu Feb  5 22:37:01 2004
Subject: [DGD] Melville under the Kernel Lib

In the past archives, Steven has stated if he were to "do it again" he'd
prolly choose to go on top of the kernel. Yet in quite a few posting's also
has stated a 'distaste'(?) for the directory depth's of such. 

Yet there doesn't seem to be any way to accomodate (what we've been calling)
the Klib without "breaking" its directory requirements, and making it innured
to any future patches from the experimental line of DGD.

So a question for Mr.moby and others that are either building ontop of the
Kernel and/or "breaking" the kernel. How doable would changing that spec
around be to something like:
/lib/kernel/
/obj/kernel/
/sys/kernel/

and if we look at Noah's additional usr's like "common"
/lib/common/ etc...
and perhaps putting "/usr/System/" into non-subdirectoried
/lib/, /obj/, /sys/

For the purposes of "file Security/inheritance/clone/update/et al" I think
that would work. What I'm not sure about atm, would be the feasibility of
"resource management" which for coders themselves in a /home/&lt;username&gt;,
would be easy enough to follow the current Klib design. But not sure about
resource management of non Coder/wizard's under this route.

Would it be worth the restructuring? Or should one just bite the bullet and
work under the Nazi Klib file regime ;)

The few posts of Skoto's that referred a filepath would seem to indicate a
non-KernelLib obeying structure.  So what have others done in this regard?
And how might you structure Steven? 
I didn't much care for the Klib design, but least I understand it more now,
but the other admin is fairly adamantly against it, which is leaving us at a
bit of an impass I'm not sure how to resolve without possibly too much
unneccessary work at 'breaking' the klib to bring in features we'd just like
the "Melville-like" directory structure to support.
Thanks, 
Zamadhi
</pre>
<hr>
<pre>
From: dgd at list.imaginary.com (Noah Gibbs)
Date: Thu Feb  5 22:59:01 2004
Subject: [DGD] Melville under the Kernel Lib

--- Michael McKiel wrote:
> Yet in quite a few posting's also
> has stated a 'distaste'(?) for the directory depth's of such. 

  Yeah.  That's my big problem with the Kernel Lib as well.

> How doable would changing that spec
> around be to something like:

  If you're going to break the current Kernel Lib, you might as well go
all the way and give it per-file or per-directory permissions, even for
non-user directories.  And in that case you can scrap a lot more of the
structure.  Basically all of it, in fact.

> What I'm not sure about atm, would be the
> feasibility of
> "resource management" which for coders themselves in a
> /home/<username>,
> would be easy enough to follow the current Klib design. But
> not sure about
> resource management of non Coder/wizard's under this route.

  The Kernel Library requires you to register all resource owners.  You
can alter the user object so that it doesn't give a wiztool to every
resource owner easily enough, though that's what it does now.

  I think registering all resource owners is a good idea.  I'd
recommend sticking with it.

> Would it be worth the restructuring?

  Not if you do it that way, no.  You're not getting much of an
improvement over the existing system.  There are *still* too many
directories.  The big problem that most people seem to have (I know I
do) is that you need to put cloneables, LWOs, daemons and inheritables
in separate directories from each other, and further segregate based on
privilege.  That's a lot of directories.  Being able to set the
permissions of LPC code per-directory would help a bit, but probably
not enough.  Being able to set the permissions per-file would reduce
the number of directories, though you still need extra obj and lib
subdirectories under the user dir.

  It would be better if you could distinguish libraries from cloneables
from LWOs in some other way, perhaps not based on filename.  But then
you'd have to worry about what happens if one becomes another, or else
make some mechanism to keep that from happening.

> The few posts of Skoto's that referred a filepath would seem to
> indicate a
> non-KernelLib obeying structure.

  Actually, they run on top of the Kernel.  However, they also have a
full non-file heirarchy that lives separately in their lib, and that
one doesn't use file paths, so it doesn't obey the Kernel Lib structure
for file paths.

> I didn't much care for the Klib design, but least I
> understand it more now,

  Yeah, it takes some figuring out.

> but the other admin is fairly adamantly against it, which
> is leaving us at a
> bit of an impass

  As him how exactly he intends to do recompiling in place.  If he
asks, "what does that have to do with it?", ask how he plans to
separate inheritables from cloneables.  When he says, "huh?", say he
should figure out what the Kernel Library does before he gets rid of
it.  At that point he'll refuse and get huffy and you'll need to find a
new partner.

  That's not the most elegant way to solve your problem, but it'll
trade it for a new and different problem :-)

> I'm not sure how to resolve without possibly too
> much
> unneccessary work at 'breaking' the klib to bring in features we'd
> just like
> the "Melville-like" directory structure to support.

  Yeah.  If you're willing to ditch recompile-in-place, you can do a
lot better.  Most of the Kernel Library's structural weirdness comes
(indirectly) from that.

  Of course, that's also one of the biggest, coolest features that DGD
gives you.
</pre>
<hr>
<pre>
From: dgd at list.imaginary.com (Stephen Schmidt)
Date: Thu Feb  5 23:04:01 2004
Subject: [DGD] Melville under the Kernel Lib

On Thu, 5 Feb 2004, Michael McKiel wrote:
> In the past archives, Steven has stated if he were to "do it again" he'd
> prolly choose to go on top of the kernel. Yet in quite a few posting's also
> has stated a 'distaste'(?) for the directory depth's of such.

Among other distastes, but we can start with that one. (I'm not
knocking the kernel lib, which is a great lib. But everyone has
their own tastes for power of code vs. simplicity of code. The
kernel lib places relatively high value on power; I place
relatively high value on simplicity.)

> Yet there doesn't seem to be any way to accomodate (what we've been calling)
> the Klib without "breaking" its directory requirements, and making it innured
> to any future patches from the experimental line of DGD.

I concur.

> So a question for Mr.moby and others that are either building ontop of the
> Kernel and/or "breaking" the kernel. How doable would changing that spec
> around be to something like:
> /lib/kernel/
> /obj/kernel/
> /sys/kernel/

Wouldn't make a lot of difference to me personally. I think there
is some advantage to having -all- kernel code in /kernel. Then you
can have a simple "Do Not Enter" rule for that directory and be OK.

> and perhaps putting "/usr/System/" into non-subdirectoried
> /lib/, /obj/, /sys/

It's been a long time since I looked at the kernel (close on
five years) but I remember /usr/System confusing the living
daylights out of me. This is probably because I do not know
much about UNIX systems, and anything that is built around
a resemblence to UNIX is opaque to me (as it is to 98% of
the population, even those who can program). I suspect if
I was a semi-serious UNIX hacker I'd find the kernel lib
much more approachable. But I'm not. Most people aren't.
(I said the same thing about MudOS and it did me no good
then.)

Melville was written to look familiar to anyone who had used
a MudOS mudlib or other things that we basically variants on
LP 3, but reducing the UNIXisms. In 1994 that was a good
strategy. Today it might not be.

> Would it be worth the restructuring?

No.

> The few posts of Skoto's that referred a filepath would seem to indicate a
> non-KernelLib obeying structure.  So what have others done in this regard?
> And how might you structure Steven?

I don't really have an opinion on that subject, other than that
the kernel lib should control what is in its own space (mostly
/kernel) and let the high-level mudlib control the rest. I didn't
like the way the kernel lib imposed a lot of structure on the
contents of /usr, particularly.

However, it's my vague understanding that a lot of that imposed
structure is necessary to support persistance, and that's an area
I do not grok, although I have some awareness of the issues. So
I don't feel knowledgeable enough to seriously comment, and
certainly not enough to propose changes.

In 1999 I started a project to produce a mudlib that would run
over the kernel but be similar in feel to Melville. After a month
I realized a) the kernel was much harder to deal with than I had
anticipated, b) the kernel imposed more requirements on the high
level lib than I had realized, and c) understanding how persistance
really worked was going to take more time than I had. I did not
begin to realize until a couple years ago just what a fundamental
difference that makes. Anyone who doesn't understand it should
probably be slow to make changes to the kernel. Anyone who does
not require persistence might be well advised not to use the
kernel if a satisfactory alternative is available (my 0.02
cents only and I'm sure many people disagree ;)

There is a chance that someday I'll go back to the kernel and
building a Melville-style lib that runs on top of it. But for
the last year and a half I've been programming Napoleonic Wars
simulation games and that's taken all the time I have. :)

Steve/Moby
</pre>
