---
title: "Frame-Dependence"
date: 2019-05-23T14:12:20-07:00
weight: 20
---

![oh dear](/images/oh_dear.gif)

Oh dear. That doesn't seem right at all.

The paddle is moving way too fast. But our speed value is only 10. What's going on?

Remember when I mentioned *frame-dependence* and *delta-time* earlier? This is a classic example of frame-dependence. Notice that the FPS, or frames-per-second, of the game is around 500 in the above recording. Our motion message when we press the "up" key on the keyboard tells the paddle to move 10 units.

That means every frame we have the "up" key held down, the paddle is moving 10 units. Which means, as things stand right now, the paddle is moving about 5000 units per second. And if the framerate changes for some reason, the paddle will move slower or quicker. This means the actual progress of the simulation will be completely different on slower or faster computers.

This is where *delta-time* comes in to save the day.

*delta-time*, as I mentioned before, is the amount of time that has passed between the previous frame and the current frame.

If we multiply the rate of change of the position by delta-time, then the paddle will move at the same speed no matter whether the framerate changes.

Let's go back to the lines of the MotionEngine where we update the position.

```ts
position_component.x += message.x * dt;
position_component.y += message.y * dt;
```

![better](/images/better.gif)

Our simulation is now *frame-independent*, which is what we desire. The paddle is definitely moving too slowly now, but that's something we can fix with a bit of value tweaking.

It is very important that you take care to multiply things that change over time by the delta-time value, or you risk elements of your game becoming frame-dependent.
