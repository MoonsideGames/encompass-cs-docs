---
title: "ECS"
date: 2019-05-21T15:55:45-07:00
weight: 25
---

ECS stands for **Entity-Component-System**. It is based on two fundamental principles:

* There should be complete separation between data and logic.
* Objects should be created via *composition* and not inheritance.

**Components** are the most basic element. They are simply containers of related data. In a 2D game we could have a PositionComponent with an *x* and *y* value. Components do not contain logic, though they might have callbacks to deal with side effects, for example creating or destroying bodies in a physics simulator.

**Entities** are generic objects, which have a unique ID and a collection of Components. Entities have no inherent behavior or properties of their own, but are granted these by Components. We can add, remove, or modify Components of Entities as necessary during the simulation.

**Systems** are responsible for reading and modifying Entities and Components. For example, a MotionSystem might look at Entities that have both a PositionComponent and VelocityComponent, and update the PositionComponent based on the information in the VelocityComponent.

Notice, in our above example, that this gives us a tremendous amount of flexibility. We can completely remove the VelocityComponent from an Entity while the game is running, and the Entity will simply stop moving - nothing else will break, because nothing has to rely on an Entity specifically containing a VelocityComponent! This is the power of composition and de-coupling at work.

Unfortunately, ECS is not without its problems.

Suppose we have a game where the player character can move left or right, and also has a special ability that lets them teleport forward over a short distance. We could put all of this logic in the same System, but most Entities that move around are not going to have this behavior, so it makes sense to have a separate system. In other words, we have introduced multiple systems that need to manipulate the position of objects.

We could have a TeleportSystem that immediately sets the *x* and *y* value of the PositionComponent so it is in front of the player's current position. But if the TeleportSystem runs before our regular Motion System, the Position Component will be overwritten by the regular Motion System and we will not see the teleportation behavior occur, even though the system is running!

This kind of bug is known as a *race condition*. Race conditions are one of the worst kinds of bug to handle, because it is incredibly hard to trace values being over-written by other values. In my experience, race conditions are probably a good 80% of what game programmers deal with. No wonder we are so frustrated all the time!

In my opinion, being forced to think about the order in which Systems run is a form of tight coupling, because it means you have to consider the behavior of other systems when writing a new system to avoid introducing bugs.

This example also illustrates another problem with ECS: in Systems that have similar behavior, we don't really have a nice structure to share that behavior.

I have created a variant of ECS that intends to address these and other problems. Let me tell you about it.
