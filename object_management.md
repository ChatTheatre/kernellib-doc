---
title: Object Management
---

This article is a stub.

## Object Managers

The Kernellib permits your application to register an object manager. Especially if your application is persistent, it should definitely do so.

An object manager keeps track of allocated objects and can notify those objects when they are upgraded. This is very useful if you plan to keep data while updating the code for a given object.
