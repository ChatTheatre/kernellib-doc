---
title: Permissions
---

(This article is a stub.)

## Security

Most of the security provided by the klib is enforced by functions in the auto object masking kfuns in dgd, such as anything to do with files or objects.

One key part of an object is its creator, which is the owner of the file the object was compiled from.  Creator determines many privileges.

The System creator has unique privileges from the klib, including being able to inherit libraries from the kernel library (apart from the auto object of course), call certain functions in the klib, and override certain privilege checks that apply to other creators.

Please refer to the kernel library's documentation for further information.

Creators and owners can be real people, or game modules.

The admin user is given the ability to back-door using the first binary port listed in the config file, provided their password in the klib shell (located in /kernel/data) is enabled.

## Ownership

All objects in the klib are owned.  Master objects are owned by their creators, while clones are owned by the object that cloned them.

The klib forbids kernel objects from being cloned or destructed except by other kernel objects, and forbids objects from destructing other objects that do not have the same owner, unless the destructing object has the System creator.
