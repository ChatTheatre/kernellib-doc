---
title: Resource Management
---

This article is a stub.

## Resources

Many things are optionally limited.  Ticks, stack space, callouts, objects, and other things can be given a quota which is enforced on a creator-wise basis by the klib.

### Rlimits

It is possible to impose runtime limits on stack or tick usage by means of the rlimits construct.

rlimits constructs are evaluated by the driver object both when the program using them is compiled, and if necessary, also during runtime.  A denial by the driver on an rlimits attempt will cause a runtime error.

Exhausting the stack limit will cause a runtime error.
