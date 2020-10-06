---
title: Tracking and Dumping Objects
---

If you have a lot of privilege on an existing DGD-based application (e.g. [SkotOS](https://github.com/ChatTheatre/SkotOS),) one great difficulty can be inspecting the LPC-based contents of memory.

DGD permits objects with older code to persist in memory across reboots. This can present a significant challenge to the would-be explorer.

Familiarity with the [object manager](object_management.md) can help significantly here.

## Object Dumping

The more privilege your code has, the more objects it can successfully inspect.

In the best case, you may have the ability to create new objects under /usr/System or even modify the Kernellib itself. Since modifying Kernellib is normally a bad idea, let's talk about how to dump objects via /usr/System.

Here's an example of how to traverse all objects based on Shentino's [skoot/usr/System/sys/tool/gcollect.c](https://github.com/ChatTheatre/SkotOS/blob/master/skoot/usr/System/sys/tool/gcollect.c):

```
/* /usr/System/sys/objtraversed.c */

#include <kernel/kernel.h>
#include <kernel/rsrc.h>
#include <kernel/objreg.h>
#include <config.h>

inherit rsrc API_RSRC;
inherit oreg API_OBJREG;

static void create()
{
    rsrc::create();
    oreg::create();

    call_out("traverse", 0);
}

static void do_something_with_object(string tracked_owner, object obj)
{
    /* This is what you actually want to do, such as examine or dump the object. */
    /* The code as written above will call this method on nearly every LPC object
       on your entire server. You probably want to filter more carefully than that for
       any traversal you will do repeatedly. */

    string obj_owner;

    /* This is an example check. Substitute your own. */
    obj_owner = obj->query_owner();
    if(tracked_owner != obj_owner) {
        DRIVER->message("Owner " + (obj_owner ? obj_owner->name() : "Ecru") + " of object " +
            obj->name() + " does not matched tracked owner " + tracked_owner->name() + ".\n");
    }
}

static void traverse_owner(string owner, object first, object obj, string obj_name)
{
    object next;

    if (first_link(owner) != first) {
        DRIVER->message("Owner '" + (owner ? owner : "Ecru") + "': list changed, aborting scan.\n");
        return;
    }

    if (!obj) {
        DRIVER->message("Owner '" + (owner ? owner : "Ecru") + "': lost iterator at item " + obj_name + ", aborting scan.\n");
        return;
    }

    next = next_link(obj);

    do_something_with_object(owner, obj);

    if (next != first) {
        call_out("traverse_owner", 0, owner, first, next, next->name());
    } else {
        DRIVER->message("Owner '" + (owner ? owner : "Ecru") + "': list traversed.\n");
    }
}

static void traverse()
{
    string *owners;
    int sz, i;

    owners = query_owners();
    sz = sizeof(owners);

    for (i = 0; i < sz; i++) {
        object first;
        string owner, first_name;

        owner = owners[i];
        first = first_link(owners[i]);
        first_name = first ? first->name() : "no object";

        DRIVER->message("Owner '" + (owner ? owner : "Ecru") + "': list traversal started with " + first_name + ".\n");
        call_out("traverse_owner", 0, owner, first, first, first_name);
    }
}
```

To use this, copy the above code into an appropriate place under usr/System/sys inside the [DGD root](directories.md). Perhaps you could call it objtraversed.c?

You'll want to compile it via something in /usr/System. If you cold-boot your application, add a line like `compile_object "/usr/System/sys/objtraversed"` to /usr/System/initd.c.

As written, it will traverse your objects when it's compiled because traverse gets called from create(). You could also remove that line and call it later, perhaps with a wiztool command like `code /usr/System/sys/objtraversed->traverse()`.

Keep in mind that your traversals may take some time to complete, especially if you have a lot of objects. This will use call_out to make sure you have plenty of ticks (CPU) for your traversal, but it can take a long time to complete fully &mdash; especially if any single owner has a lot of objects.
