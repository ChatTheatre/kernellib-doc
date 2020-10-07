---
title: Security
---

The Kernellib has strong ideas about what operations are permitted when, and to whom.

## What Matters?

When a program /usr/foo/obj/bar is trying to do a particular thing (call another object? compile an object? read a file?), several things are important.

Two of these are the creator and the owner of the object. It can also matter whether the object is a clone or an instance (as opposed to a library, say.)

For anything under /usr/foo, the creator is "foo". For anything under /kernel or /usr/System, the creator is "System."

For clones, the owner is whoever is cloning the object. For libraries and cloneables, the owner is the creator.

The caller (what object is calling the given method) can also matter. When we talk about, e.g. "kernel callers" we mean that the previous_program() calling this method must also be in a kernel object.

## Access

Another thing that can matter is whether a particular user has read- or write-access to a given directory. Certain directories can be marked for universal read access (that is, anybody can read them.) A given user can also be given individual access to particular bits of the file hierarchy.

## Tilde-paths

If a path has a tilde in it, that will expand relative to the object creator's home directory.

## find_object()

You can only find an object from a non-destructed other object. You're not allowed to find a library object because you're not allowed to have a reference to one.

## destruct_object()

Libraries can be destructed pretty easily in general. For non-libraries:

You can only destruct an object from a non-destructed calling object.

Only kernel callers can destruct kernel objects.

Objects with a non-System creator can only be destructed by objects with the same owner.

## compile_object with source given

Compile_object is allowed to receive source code, which becomes the source of the new object. But that's not permitted for objects under /kernel, ever.

You can only compile an object from a non-destructed calling object.

## compile_object with no source given

You can only compile an object from a non-destructed calling object.

Objects with non-System creators can only be compile an object if the compiling object has (mostly) write access to the object being compiled. In a few specific cases (e.g. libraries or creator-less objects being compiled) read access is sufficient.

Rsrcd restricts the number of objects for each creator. This can sometimes prevent an object from being compiled if that creator is "full."

## clone_object

You can only clone an object from a non-destructed calling object.

The creator of the cloning object must be System.

Only Kernel callers can clone /kernel objects.

Objects with non-system creators need read access to clone an object.

Only objects with an owner can clone other objects.

Only compiled objects can be cloned.

Lightweight and inheritable objects (those with /lib/ or /data/ in their path) cannot be cloned.

Only objects with /obj/ in their path can be cloned.

## new_object

TODO

## call_trace

If the object calling call_trace has a non-System creator, some parts of the call stack will not be visible.

## swapout, dump_state, shutdown, call_touch

Only objects whose creator is "System" can trigger these operations.

## call_out

Only a non-destructed function can schedule a call_out.

Kernel objects can schedule direct call_outs without the Kernellib as an intermediary.

Callouts are tracked as a resource, and can only be scheduled if enough are left for the owner.

## add_event

Persistent objects (e.g. clones) can add events, while LWOs cannot.

## read_file

A destructed object may not read a file.

Objects with a non-System creator need read access to a file's path to be allowed to read the file.

## write_file

A destructed object may not write a file.

Objects with a non-System creator need write access to a file's path to be allowed to write the file.

Nobody may write to /kernel or /include/kernel.

A given creator may not write files when over file quota.

## remove_file

A destructed object may not remove a file.

Nobody may remove files under /kernel or /include/kernel.

Objects with a non-System creator need write access to a file's path to be allowed to remove the file.

## rename_file, get_dir, file_info, make_dir, remove_dir

TBD

## restore_object

Objects with non-System creators need write access to an object's path to be allowed to restore to it.

## save_object

Objects with non-System creators need write access to an object's path to be allowed to save it.

Also, filequota is checked.

## editor

TBD

## Driver->message

Only /kernel and /usr/System objects can write to console via driver->message.

## Object inheritance

An object needs read access to a library to be allowed to inherit it.

If an object manager is registered it may use forbid_inherit to impose additional conditions.

## include_file

An object needs read access to another file to be allowed to include it.

If an object manager is registered it may use include_file to impose additional conditions or otherwise affect the inclusion.

## remove_program, recompile

These are protected from being called by random code (note: how? I've checked experimentally, but not sure why it works.)
