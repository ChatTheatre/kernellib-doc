---
title: Adding to your MUDLib
layout: default
---

## Adding Stuff to your MUDLib

So you've got a good start on a MUDLib, either by [modifying the Kernel MUDLib](Modifying.md) directly or with
the help of Shevek's BBLib.
You're rip-roaring to go and you've even been adding a few little
extra features here and there. Great! Welcome to the ever-expanding
community of DGD MUDLib authors. We're happy to hear from you on
the mailing list, we're happy you're out there, and we look forward
to your contributions to the set of MUDLibs we can study and
use!

What can we do to help you? Well, for starters we can suggest
some things you should do to keep your MUDLib usable, featureful
and secure -- and believe me, doing all that at once is
*hard*! The **Adding Stuff to your MUDLib** section on the
previous page is here to help you figure out some things you should
put in your MUDLib. It can't tell you everything. The things that
make your MUDLib different and special are the things we'd never
think of because only <i>you</i> have the vision to make them
happen. But we can help you fix some common problems and avoid some
common pitfalls.</p>

## About "Adding Stuff"

In this series of documents we'll try to talk about security
pretty regularly. We tend to assume that you're going to have some
not-really-trusted people running code. That may be your users but
more often it'll be admins or builders you've given access to so
they can build areas. They may or may not be fully trustworthy but
even the best programmer can have a bad day now and again. When
that happens it's best if your MUDLib is robust enough to handle
it. It can make sure that the worst they get is a nasty error
message on their command line while the MUD chugs along with nary a
hitch for the other users. It's not really possible to make that
happen <i>all</i> the time, especially with your highest-level
admins, but we can keep you from tripping over the wrong thing
quite a lot of the time if you're willing to code carefully. DGD is
just that cool.

That also means that if you're only going to give coding rights
to people you absolutely trust and who would be allowed to bring
your entire MUD crashing down around your ears and destroying all
your data if they felt like it... Well, then you can ignore pretty
much all our security advisories. Your MUDLib won't be the first to
do so.
