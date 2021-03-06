---
title: Layers of DGD
layout: default
---

## The DGD Driver and the basics of the MUDLib

The DGD driver presents a series of layers of functionality.
What kind of stuff <i>you</i> think DGD does depends on what layer
you see. Lets briefly dissect the layers between the DGD Kernel
MUDLib and the OS you're running on.

At the lowest, scariest, hardwariest level that you can still
call DGD, you have the DGD driver itself. The DGD driver is written
in C, and is an interpreter for a language called LPC that looks
kinda similar, but not <i>very</i> similar. If you download DGD
from the official web site, this is what you're compiling, probably
after applying a lot of patches. It's not very well documented, but
it <i>is</i> very solidly written.

You can just directly hack the source code if you don't care
about being compatible or getting Dworkin's updates. That means
hacking DGD just like any other application on your OS of choice.
If you don't <i>fully</i> understand what I mean by all this or if
you don't have much experience writing programs on the machine
you're sitting in front of, then this isn't the level you want to
work at. Even if you do, it probably still isn't.

DGD has an interface for <b>extensions</b>, which are pieces of
code that make the driver do new things without breaking the old
ones. This is <i>still</i> a lot like hacking the driver, and it
can still cause big-time problems. If you write your code this way
and you <i>ever</i> have to change it, you'll need to bring your
server down to fix it. That means you're ruining some of that
persistence that DGD works so hard to give you. This is something
you'll have to do in a few cases, but mainly you should leave it
well enough alone.

The driver and extensions (mentioned above) provide various DGD
kernel functions, also called kfuns. These are the functions that
the lowest-level MUDLib code uses to present more interesting
services to your LPC code. Kernel functions do things like allocate
memory, generate random numbers, give security information, return
basic driver constants and do network I/O. They don't look much
like a game, MUD or otherwise. In fact, it's more like a little
Operating System and its system calls. Here's the list of them as
of DGD 1.2.71:

<pre>
acos            call_other       find_object      new_object        sin
allocate        call_out         floor            object_name       sinh
allocate_float  call_touch       fmod             parse_string      sizeof
allocate_int    call_trace       frexp            pow               sqrt
asin            ceil             function_object  previous_object   sscanf
asn_add         clone_object     get_dir          previous_program  status
asn_and         compile_object   hash_crc16       query_editor      strlen
asn_cmp         cos              hash_crc32       query_ip_name     swapout
asn_div         cosh             hash_md5         query_ip_number   tan
asn_lshift      crypt            hash_sha1        random            tanh
asn_mod         ctime            implode          read_file         this_object
asn_mult        decrypt          ldexp            remove_call_out   this_user
asn_or          destruct_object  log              remove_dir        time
asn_pow         dump_state       log10            remove_file       typeof
asn_rshift      editor           make_dir         rename_file       users
asn_sub         encrypt          map_indices      restore_object    write_file
asn_xor         error            map_sizeof       save_object
atan            exp              map_values       send_datagram
atan2           explode          millitime        send_message
block_input     fabs             modf             shutdown
</pre>

If you use an LPC driver other than DGD (such as MudOS) you'll
find that these functions are very different. Another driver's
functions will look more like the basics of a game and less like an
operating system. DGD is designed to work as pretty much any kind
of network server, not just as a MUD. People have written servers
for HTTPD in DGD, and Yahoo Chat runs on top of it. All of that
means that DGD MUDLibs have to do a lot more to give a game
environment.

Of course, that also means that if you want to make a MUD that
doesn't act at all like traditional MUDs then DGD is the driver for
you -- it doesn't force you to use any of the regular MUD functions
that you no don't need or want. So if you want a MUD with no object
or room data structures at all (maybe you just have particles
everywhere and you figure it all out from that!), DGD provides you
a simple, clean interface with nothing extra to trip you up or slow
you down.

Above the level of the DGD driver kfuns is your MUDLib, written
in LPC. Your LPC will translate the raw kfuns above into something
you can use more easily. However, the kfuns above can require a lot
of supporting infrastructure, and that's where the DGD Kernel
MUDLib can help you. It sits above the kfuns and below your own
MUDLib.

You may find that if you try to write code that uses the kernel
functions above in the way they're documented by DGD that you can't
-- you may be denied permission, or you may even be told you're
calling with the wrong number of arguments. That's because the only
part of a MUDLib that can directly use DGD's driver kfuns are
special objects called the driver object and the auto object. If
the auto object decides to change how a function works then all the
other code gets their version, not version from the DGD driver.

The Kernel MUDLib overrides a number of these so that it can
perform more security checking, which is good for the MUD
administrator -- security can keep the builders and other wizards
from being able to do Bad Things, at least if you use it
consistently and intelligently.

The Kernel MUDLib, like basically every MUDLib, overrides some
of these and provides new ones. The Kernel MUDLib <i>still</i>
doesn't look very much like a MUD because it's really designed to
let you build a MUDLib on top of, not to be the primary MUDLib for
your game. Some MUDLibs, such as the 2.4.5 MUDLib and Melville,
provide you more of a game. With the Kernel MUDLib, you'll need to
build it. That's okay. The Skotos folks do just exactly that,
so do I, and you can too.

In fact, for everything I'm writing here, I'm assuming you'll be
building on top of the Kernel MUDLib. That'll mean that if you're
using Melville or 2.4.5 there will be differences, and if you're
just building on top of raw DGD there'll be even more. Those are
all fine places to start, but in <i>this</i> document it's all
Kernel.

And that's why you're here. At least, that's why you're reading
this page. It's because you want to write a MUD and you're brave
enough to start from the best around (DGD and the kernel MUDLib)
even if that's much harder than using something that's already
written and polished for you. Congratulations!
<hr>

## DGD Kernel Functions

The kernel functions are documented in your DGD distribution in
dgd/doc/kfun. Second, for whatever MUDlib you're using, these
documents may be inaccurate. Read the overview above to find out
why DGD's raw kernel functions may not be the same ones that you
use from the Kernel MUDLib.

So first, we should have a look at the functions that the Kernel
MUDLib overrides and the new ones it provides. You can find it for
yourself in directories under "dgd/mud/doc/kernel", but here's a
quick list, up-to-date for 1.2.71:

<pre>
Lfuns:
allow_subscribe  create  query_owner

Efuns:
add_event     compile_object   find_object             remove_event
call_limited  destruct_object  get_dir                 status
call_other    event            new_object              subscribe_event
call_trace    event_except     query_events            unsubscribe_event
clone_object  file_info        query_subscribed_event
</pre>

If you're paying attention, you'll have figured out that that
means the functions the Kernel MUDLib actually overrides are:

<pre>
call_other            compile_object         get_dir
call_trace            destruct_object        new_object
clone_object          find_object            status
</pre>

The other ones are all new.

There are also special interfaces called hooks. Certain methods,
such as set_object_manager, can be called on particular objects and
later on, that object will call functions within the object manager
you provided it. That means it needs to call functions it knows all
about, so you'll need to have given it an object manager with all
the functions it needs. That's how hooks work. Luckily, the Kernel
MUDLib documents the different hooks and what kind of object you'll
need to give them pretty well. You can find the documentation in
mud/doc/kernel/hook.
