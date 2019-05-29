---
title: "Bouncing"
date: 2019-05-28T18:47:18-07:00
weight: 100
---

Let's make the ball bounce off the sides of the game window.

I know what you're thinking. "Let's just read the dimensions of the game window. When the ball goes past them, we know it should bounce!"

**NO.**

We don't want the behavior of any object to be directly tied to some state outside of the game simulation. That's just asking for trouble!! What if you want the game boundaries to be different from the window size later? What if a different device has different dimensions? The possibilities are endless!

There's something else going on here too: eventually we're gonna need the ball to bounce off of the paddles as well right? I think what we really need here is a collision system.

All of our objects are rectangles so I think a simple AABB (axis-aligned bounding box) check will suffice. Essentially we just use non-rotating rectangles and check if they are overlapping.

Hang on a sec though - this is a pretty standard problem right? Pretty much every game in existence uses collision detection.

LOVE provides a physics system under _love.physics_. But it's actually a fully featured physics simulator that attempts to behave realistically under physical constraints. It's a little heavy to use it just for simple collision detection and we'd probably have to rework all of our entities and components to integrate it properly at this point.

Turns out that there is a library for AABB collision that will work just fine for our purposes. It's called [bump.lua](https://github.com/kikito/bump.lua). Let's start by integrating it.
