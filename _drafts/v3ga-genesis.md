---
layout: post
title: V3GA Genesis
tags: v3ga
---

*This is part of the [V3GA Project][v3ga-overview].*

## What's in a name?

I haven't posted here in a while, but rest assured that I've still been mulling
over the various details of the project in the mean time. The most obvious
thing that I've changed is the name. I was never particularly thrilled with
FXM, so I spent some time trying to come up with a better name.

I like V3GA for a few reasons:

* One of the core goals of the computer is to be able to drive a monitor at VGA
  resolution
* The processor will have a 3-stage pipeline
* It's tempting me to make a [splash screen][splash-screen]

## What's new?

I've started on a grand architecture document, mostly with the goal of
carefully thinking through all of the various instructions, features, and
peripherals so that I don't paint myself into any corners. It's not hosted here
yet, but at some point I'm going to convert it over to Markdown and post it.

While trying to spec out some of the various pipeline stages, I've realized
that things are going to be tricky if I have to rely on the worst-case
propagation delays of the 74F18\* ALU and carry-lookahead chips, which is
generally good engineering practice. I might be able to make it work if I track
down some really fast OR and NAND gates to make a custom carry calculator for
the first half of the ALU, but that won't leave me any margin for any other
calculations, like generating condition codes.

Rather than trying to design some clever time-borrowing mechanism, I've decided
to develop a test board which will allow me to actually characterize the
propagation delays of some test chips.

[v3ga-overview]: /v3ga-overview
[splash-screen]: http://youtu.be/a9224qZ7N38
