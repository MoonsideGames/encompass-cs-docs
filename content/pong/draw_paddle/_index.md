---
title: "Drawing A Paddle"
date: 2019-05-23T11:02:45-07:00
weight: 0
---

It's nice to see something on screen right away when we start making a game, so let's make that happen.

In a 2D game, Encompass needs to know which order that things should draw in.

Encompass draws things back to front using integer layers. A negative value means farther in the back. A positive value means farther in the front. So an object on layer 10 will draw on top of an object on layer -10.

We'll need two things to get a paddle drawing on screen: A *DrawComponent* and an *EntityRenderer*.

Let's start with the Component.
