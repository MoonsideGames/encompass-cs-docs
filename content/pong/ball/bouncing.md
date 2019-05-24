---
title: "Bouncing"
date: 2019-05-23T18:38:51-07:00
weight: 30
---

Let's make the ball bounce off the sides of the game window.

I know what you're thinking. "Let's just read the dimensions of the game window. When the ball goes past them, we know it should bounce!"

**NO.**

We don't want the behavior of any object to be directly tied to some state outside of the game simulation. That's just asking for trouble!!

Let's make a BoundariesComponent instead. In **game/components/boundaries.ts**

```ts
import { Component } from "encompass-ecs";

export class BoundariesComponent extends Component {
    public left: number;
    public top: number;
    public right: number;
    public bottom: number;
}
```

Now we have two options. We could put this on the ball or on a separate Entity. But let's think about it - the paddles need to respect the boundaries too, right? So let's make it a separate Entity.

{{% notice tip %}}
It's perfectly valid to create Entities for the purpose of only holding one Component. It's good architecture a lot of the time!
{{% /notice %}}

In **game/game.ts**:

```ts
const boundaries_entity = world_builder.create_entity();
const boundaries_component = boundaries_entity.add_component(BoundariesComponent);
boundaries_component.left = 0;
boundaries_component.top = 0;
boundaries_component.right = 1280;
boundaries_component.bottom = 720;
```

Now how do our boundaries actually work? You might recall that our MotionEngine updates the positions of objects directly. But that means it could leave something out of the boundary. Remember that we can't have two Engines mutate the same Component type.

We could do boundary checking inside of MotionEngine. But we don't know that everything that moves will need to respect the boundary, so it seems bad to put that inside of MotionEngine.

I think what we should do is have two new Engines. The first will be an Engine that takes a Message that reads a position and checks if it is outside of the boundaries. Then we can make a new Engine that finalizes the new value of the PositionComponent.

```ts

```
