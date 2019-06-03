---
title: "Scoring"
date: 2019-05-30T16:08:54-07:00
weight: 1000
---

Now we're getting into the meat of the game.

In Pong, your opponent scores a point by getting the ball behind your paddle. There's a few things we need to sort out here.

- The ball needs to detect collision with the goals.
- The ball needs to disappear for a while when a goal is scored.
- The ball position and velocity needs to be reset on a "serve".
- The score of each player needs to be tracked.

Here's what I'm thinking: first let's sort out collision detection with the goal and resetting the ball. Then we can set up a scoring mechanism.
