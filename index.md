---
layout: default
title: Kernellib
---

[Dgd](https://ChatTheatre.github.io/lpc-doc/) has [some unusual characteristics](https://ChatTheatre.github.io/lpc-doc/dgd/unusual.html). It also provides enormous flexibility. Kernellib provides control over those unusual characteristics, and provides sensible defaults on top of that flexibility.

Quick Reference:

* [Features of Kernellib](./features.html)
* [Installing the Kernellib](./installing.html)
* [Default Kernellib Commands](./commands.html)
* [Miscellaneous other documentation about the Kernellib](./miscellaneous.html)

## But What Is It?

DGD is a raw, low-level language. You can think of it as being comparable to Java or C with the system libraries. Certain common tasks that a long-running DGD application will need simply aren't part of the system libraries, just as is the case with other languages.

The [Kernellib](https://github.com/ChatTheatre/kernellib), formerly called the Kernel MUDLib, provides a number of features that a long-running persistent application, such as a game, is likely to need. By providing these features in a well-known standard way, higher-level DGD library code can be written in a way that is likely to work across many different applications.

The Kernellib is no longer the most recent library of that kind. Dworkin (DGD's author) is currently working on an expanded version of DGD called Hydra, and has a Kernellib-like library called [Cloud Server](https://github.com/dworkin/cloud-server) that is meant to serve a similar purpose.

Despite aiming to provide a more uniform interface than DGD, the Kernellib tries to maintain a lot of flexibility. Where more than one approach is reasonable (e.g. object lifecycle management,) Kernellib tries to provide hooks and APIs rather than a full implementation.
