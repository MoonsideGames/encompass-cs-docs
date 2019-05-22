---
title: "The Messy Basement"
date: 2019-05-21T15:48:10-07:00
weight: 15
---

Unexpected behavior jumps out at you constantly. You feel like you're playing whack-a-mole with bugs. You can't remember where you put anything and keeping everything sorted is a constant nightmare.

Uh oh. You're in a messy basement.

Some characteristics of the messy basement:

* Magic values, aka putting numbers directly in the source code.
* Game objects directly manipulating each other's values willy-nilly.
* Multiple objects that keep track of and depend on each other's state.

The key characteristic of this structure is that there almost doesn't seem to be any structure at all.

Shockingly, some game engines and frameworks not only fail to prevent this kind of architecture, they actually *encourage* it!

"But *I* know where everything is!" Tell that to yourself three months from now.

I don't need to say much more about this kind of architecture. It's obviously bad. Let's move on.
