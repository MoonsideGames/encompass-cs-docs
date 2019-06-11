---
title: "Design"
date: 2019-05-28T17:27:59-07:00
weight: 400
---

Now we have a way to tell when objects are colliding. Let's make something happen as a result!

First, let's think about the structure of a collision system, and how we want it to resolve collisions.

One thing we really don't want is for collision to resolve late. For example, a frame finishing with two solid objects lodged inside each other, and then fixing itself on the next frame. Even if it's only for a frame, it's enough for players to detect some awkwardness.

We also need objects to behave differently based on _what_ they collide with. For example, a ball colliding with the top boundary is going to react differently than if it collides with a goal boundary.

For this reason, I think it makes sense to define each object as having a CollisionType; in our game a colliding object is always either a ball, a wall, or a paddle. We can implement different behaviors for a ball-wall, ball-paddle, or paddle-wall collision.

You might think it would be better to determine collision behavior by placing collision behavior components on the entities. This feels awkward to me, but I could definitely be wrong about this. If you want to try doing things that way, the power is yours! Just make sure you have a good justification for it.

You might remember that we can only modify a component in one engine. So we're going to have to be deliberate about this.

Here's what I'm thinking:

- Tell things to move
- Calculate the position they would potentially move to
- Use that position to check collision
- Do stuff in response to collision, including telling things to move again
- Resolve all the movements

We already have MotionMessages. Let's keep those, but redirect them slightly. We can't read MotionMessages and then Emit them again later down the line: that would cause a cycle. So let's have a new UpdatePositionMessage.

Let's have the MotionEngine consolidate all the MotionMessages per-component, send out an UpdatePositionMessage with that final delta, but then also send out a CollisionCheckMessage for everything that has a BoundingBoxComponent.

Finally, a CollisionDispatchEngine figures out what two kinds of objects collided and emits a Message in response. We can then implement various collision resolution Engines that read each of those kinds of Messages. For example, a BallWallCollisionMessage would be handled by a BallWallCollisionEngine.

{{<mermaid align="center">}}
graph TD;
Engines(Various Engines) -->|MotionMessage| MotionEngine[MotionEngine]
MotionEngine -->|CollisionCheckMessage| CollisionCheckEngine[CollisionCheckEngine]
CollisionCheckEngine -->|CollisionMessage| CollisionDispatchEngine[CollisionDispatchEngine]
CollisionDispatchEngine -->|BallWallCollisionMessage| BallWallCollisionEngine[BallWallCollisionEngine]
CollisionDispatchEngine -->|BallPaddleCollisionMessage| BallPaddleCollisionEngine[BallPaddleCollisionEngine]
BallWallCollisionEngine -->|UpdatePositionMessage| UpdatePositionEngine
BallPaddleCollisionEngine -->|UpdatePositionMessage| UpdatePositionEngine
MotionEngine -->|UpdatePositionMessage| UpdatePositionEngine
{{< /mermaid >}}

Whew! That's a lot! You might be wondering why we're doing all this work just to get some objects reacting to each other.

The thing is: this kind of thing is the backbone of many game designs. So of course it's a bit complex! The point is that there's no point obscuring the complexity of such a system or putting off thinking about it until later. If we build our project on a shoddy foundation we will surely have problems later. Let's get a robust system in there so we don't have to fundamentally reorganize our program at a later time, when it will be more frustrating and have more potential to break things.

Let's get started.
