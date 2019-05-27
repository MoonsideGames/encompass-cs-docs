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

There's something else going on here too: eventually we're gonna need the ball to bounce off of the paddles as well right? I think what we really need here is a collision system.

All of our objects are rectangles so I think a simple AABB check will suffice. Essentially we can use the four corners of our rectangles to see if they are overlapping.

Let's revisit our BoundariesComponent. I think we can use this for collision boundaries. And since we're specifying that everything will be a rectangle, we can simplify our BoundariesComponent to just have width and height values. In **games/components/boundaries.ts**

```ts
import { Component } from "encompass-ecs";

export class BoundariesComponent extends Component {
    public width: number;
    public height: number;
}
```

Let's write a CollisionCheckEngine. In **games/engines/collision.ts**

```ts

```

I think what we should do is have two new Engines. The first will be an Engine that takes a Message that reads a position and checks if it is outside of the boundaries. Then we can make a new Engine that finalizes the new value of the PositionComponent.

```ts

```
