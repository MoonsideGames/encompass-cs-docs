---
title: "Engine"
date: 2019-05-22T13:01:28-07:00
weight: 15
---

An Engine is the Encompass notion of an ECS System. Much like the engine on a train, your Engines make the simulation move along.

{{% notice notice %}}
I never liked the term System. It is typically used to mean structures in game design and I found this confusing when discussing code implementation vs design.
{{% /notice %}}

Engines may read any Entities and Components in the game world, read and send messages, and create and update Entities and Components.

An Engine which Reads a particular message is guaranteed to run *after* all Engines which Send that particular message.

To define an Engine, extend the Engine class.

Let's say we wanted to allow Entities to temporarily pause their motion for a specified amount of time. Here is an example Engine:

```cs
using Encompass;

[Reads(typeof(PauseComponent))]
[Receives(typeof(PauseMessage))]
[Writes(typeof(PauseComponent))]
public class PauseEngine : Engine
{
    public override void Update(double dt)
    {
        foreach (var (pauseComponent, entity) in ReadComponentsIncludingEntity<PauseComponent>())
        {
            var timer = pauseComponent.timer;
            timer -= dt;

            if (timer <= 0)
            {
                RemoveComponent<PauseComponent>(entity);
            }
            else
            {
                SetComponent(entity, new PauseComponent { timer = timer });
            }
        }

        foreach (var pauseMessage in ReadMessages<PauseMessage>())
        {
            SetComponent(pauseMessage.entity, new PauseComponent
            {
                timer = pauseMessage.time
            });
        }
    }
}

```

This engine deals with a Component called a PauseComponent. PauseComponent has a timer which counts down based on delta-time. When the timer ticks past zero, the PauseComponent is removed. If a PauseMessage is received, a new PauseComponent is attached to the Entity specified by the message.

Notice that this Engine doesn't actually "do" the pausing, or even care if the Entity in question is capable of movement or not. In Engines that deals with movement, we can check if the Entities being moved have PauseComponents and modify how they are updated accordingly. This is the power of de-coupled logic at work.
