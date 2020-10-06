---
title: Object Management
---

Object management is one of the more complex topics in raw DGD. Nearly everybody solves it by using somebody else's object manager, possibly with modifications.

Your DGD application almost certainly needs an object manager. It's unlikely that your best choice is to write one from scratch.

Even if you use somebody else's, though, it's useful to know what object managers do and what your choice will affect.

## Object Managers

The Kernellib permits your application to register an object manager. Especially if your application is persistent, it should definitely do so.

An object manager keeps track of allocated objects and can notify those objects when they are upgraded. This is very useful if you plan to keep data while updating the code for a given object.

Raw DGD doesn't implement object management. Instead, the Kernellib acts as a layer over DGD, and the object manager described here is a Kernellib convention. Put another way, if you didn't use the Kernellib you could choose an entirely different sort of API here with entirely different constraints.

### Counting and Auditing Objects

While it's possible for your object manager to simply make lists of objects, it may not need to do so &mdash; the Kernellib does some of that already.

In a Kernellib app, every object you create is part of a per-owner linked list of objects. There are "next" and "prev" fields built into every instance and clone (libraries can't be instantiated and have no data.) They are used to keep a per-owner list of all objects by the Kernellib's ObjRegD.

An object in your application under /usr/System may inherit the ObjReg API library. Any object that inherits that API can use the `first_link(owner)` inherited function to get the first object owned by each owner, and the next_link and prev_link inherited functions to traverse the linked list. The rsrc API can be inherited to (among other things) get a list of owners.

The highly-privileged objects of the Kernel library might not be subject to this tracking system. You should not assume you will find literally every Kernellib object even if you query the full list for every owner. There are a few ways to reliably query every object (e.g. binary parsing of statedump files.) But LPC cannot be fully guaranteed to reliably track LPC, particularly in the presence of bugs over time. Have you been ***fully*** consistent in ***never*** creating objects where running out of ticks might cause errors in tracking? I haven't. The Kernellib does not make all of its operations atomic by default. It is nearly inevitable that some way to track unreturned objects, or create untracked objects, exists.

For this reason you should consider additional tracking and/or auditing and/or reloading facilities on top of DGD and the Kernellib. Being able to dump all the objects you know about and reload them is one way to guarantee that ancient, un-upgraded, un-tracked objects ***cannot*** possibly exist.

### Security: include_file, forbid_call, forbid_inherit

The object manager can also implement a security model for what LPC code is allowed to inherit, call or include other LPC code &mdash; see include_file, forbid_call and forbid_inherit in the API for details.

The Kernellib implements some restrictions automatically, even if you don't ask for any extras. For instance, only an object under a /lib/ directory can be inherited. An object under a /lib/ directory cannot be instantiated, called-to or generally treated as a normal object. You must have read access to a given library in order to inherit from it. Only an object with a creator of "System" may inherit from the Kernel library itself.

If you're curious about Kernellib security, the code to its Driver and Auto objects are fairly readable. Better yet, their code implements these limitations and so they are definitive. Any odd behaviour or bugs you might find in their code are odd behaviour or bugs that all Kernellib applications are bound by.

### Touch, call_touch and Data Upgrades

An object that exists for a long time will need some form of maintenance or upgrading of its data over the years. Internal fields might be changed, for example. In any case, the older objects will need to be upgraded before they can be used by code that expects the new structure and values.

When the DGD kfun call_touch is called on an object, that object is treated as needing an upgrade (a "touch.") Specifically, before the next time that object is called, the Driver will receive a call to touch() with that object and the function to be called as arguments.

The Kernellib driver will pass that call to touch() to your object manager, which may then deal with it.

The idea is that your object manager may implement some form of dynamic upgrade before calling the function in question. A common example is that your objects may be allowed to implement a function called something like "touch" or "upgraded" to be run after their compilation, but before their next use.

Why not just upgrade everything immediately when it's compiled? A large application may have a lot of objects. An upgrade to a fundamental data field might require touching nearly (or literally) every LPC object, which can take a long time. By upgrading each object just before use, you can spread that load over a much longer period of time.

One possible worry here: if seldom-used objects don't get upgraded, you can run into a situation where an older object is ***never*** upgraded before you need another upgrade and change its touch/upgraded method and it loses the ability to ever be upgraded. To avoid this situation, it can be useful to have some ability to make sure every LPC object is "touched" (has a function called on it) at some low frequency, perhaps daily or weekly, so that no object can go longer than that without an upgrade. This could be done by literally touching every object, or by keeping a "call_touch registry" in your library to make sure that every object has been touched within some time period of the initial call_touch.

### Raw API

To tell the Kernellib to use an object manager, an object under /usr/System may call set_object_manager() with the new object manager as its only parameter. That new object manager will be notified (via called functions) whenever object-lifecycle events occur. Here are the calls it will receive from Kernellib:

```
* void compiling(string path)

    The given object is about to be compiled.

* void compile(string owner, object obj, string *source,
               string inherited...)

    The given object has just been compiled.  If the source array is not
    empty, it was compiled from those strings.  Called just before the
    object is initialized with create(0).

* void compile_lib(string owner, string path, string *source,
                   string inherited...)

    The given inheritable object has just been compiled.  If the source
    array is not empty, it was compiled from those strings.

* void compile_failed(string owner, string path)

    An attempt to compile the given object has failed.

* void clone(string owner, object obj)

    The given object has just been cloned.  Called just before the object
    is initialized with create(1).

* void destruct(string owner, object obj)

    The given object is about to be destructed.

* void destruct_lib(string owner, string path)

    The given inheritable object is about to be destructed.

* void remove_program(string owner, string path, int timestamp, int index)

    The last reference to the given program has been removed.

* mixed include_file(string compiled, string from, string path)

    The file `path' (which might not exist) is about to be included by
    `from' during the compilation of 'compiled'.  The returned value can be
    either a string for the translated path of the include file, or an
    array of strings, representing the included file itself.  Any other
    return value will prevent inclusion of the file 'path'.

* int touch(object obj, string function)

    An object which has been marked by call_touch() is about to have the
    given function called in it.  A non-zero return value indicates that the
    object's "untouched" status should be preserved through the following
    call.

* int forbid_call(string path)

    Return a non-zero value if `path' is not a legal first argument
    for call_other().

* int forbid_inherit(string from, string path, int priv)

    Return a non-zero value if inheritance of `path' by `from' is not
    allowed.  The flag `priv' indicates that inheritance is private.
```

### Implementations

There have been a few different object manager implementations over the years, done in different ways.

* [Geir Harald Hanson](https://geir-hansen.com/) wrote one (note: is this code now lost?)
* [Noah Gibbs wrote one for Phantasmal](https://github.com/dworkin/phantasmal/blob/master/mudlib/mud/usr/System/sys/objectd.c)
* [SkotOS wrote one for its games](https://github.com/ChatTheatre/SkotOS/blob/master/skoot/usr/System/sys/progdb.c).

## Old Mailing List Archive Messages on Object Management

[Here they are](phant/ObjectManager.md).
