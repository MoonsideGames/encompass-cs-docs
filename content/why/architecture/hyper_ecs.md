---
title: "Hyper ECS"
date: 2019-05-21T15:56:13-07:00
weight: 30
---

Hyper ECS is a new architecture pattern that attempts to address some common issues with standard ECS. This is the architecture that Encompass implements.

The core of the architecture is the introduction of a new construct to ECS: the **Message**.

A Message is fundamentally a variant of Component, in that it only contains data. But, it is designed to be temporary and is discarded at the end of each frame. It is used to communicate useful information between Systems.

We also introduce some extra information to Systems. Each System must declare the Messages that it **Reads**, the Messages that it **Emits**, and the Components that it **Mutates**.

Let's go back to our earlier example.

We have TransformComponent, which contains position and orientation data, and VelocityComponent, which contains an *x* and *y* component for linear motion.

Our MotionDetecterSystem reads each Entity that has both a TransformComponent and a VelocityComponent, and emits a MotionMessage, which contains a reference to the specific TransformComponent and the *x* and *y* velocity given by the VelocityComponent.

We also have a TeleportSystem that needs to teleport the character forward a bit. Let's say when the player presses the X button, a TeleportMessage is fired. The TeleportSystem reads this message and emits a MotionMessage in response.

Now we have our MotionSystem. The MotionSystem declares that it Mutates the TransformComponent, reads the MotionMessages that apply to each TransformComponent, and applies them simultaneously, adding their *x* and *y* values to the TransformComponent. Voilà! No race conditions! And we can re-use similar behaviors easily without re-writing code by consolidating Messages.

You might be wondering: how does the game know which order these systems need to be in? Well...

**Hyper ECS figures it out for you.**

That's right! With the power of graph theory, we can construct an order for our Systems so that any System which Emits a certain Message runs before any System that Reads the same Message. This means, when you write behavior for your game, you *never* have to specify the order in which your Systems run. You simply write code, and the Systems run in a valid order, every time, without surprising you.

Of course, to accomplish this, there are some restrictions that your Systems must follow.

Systems are not allowed to create message cycles. If System A emits Message B, which is read by System B which emits Message C, which is read by System A, then we cannot create a valid ordering of Systems. This is not a flaw in the architecture: A message cycle is simply evidence that you haven't quite thought through what your Systems are doing, and can generally be easily eliminated by the introduction of a new System.

Two separate systems are not allowed to Mutate the same Component. Obviously, if we allowed this, we would introduce the possibility of two Systems changing the same component, creating a race condition. If we have two Systems where it makes sense to change the same Component, we can create a new Message and System to consolidate the changes, and avoid race conditions.

If you are used to programming games in an object-oriented way, you will likely find the ECS pattern counter-intuitive at first. But once you learn to think in a Hyper ECS way, you will be shocked at how flexible and simple your programs become.
