---
title: "Moving"
date: 2019-05-23T18:10:17-07:00
weight: 20
---

We already have MotionMessages and a MotionEngine. So it seems logical to re-use these structures for our ball.

What is actually going to be sending out the MotionMessages?

What is the main characteristic of the ball in Pong? That's right - it is continuously moving. In other words, it has velocity.

Let's make a VelocityComponent. In **game/components/velocity.ts**:

```ts
import { Component } from "encompass-ecs";

export class VelocityComponent extends Component {
    public x: number;
    public y: number;
}
```

Let's also create a VelocityEngine.

What does our VelocityEngine actually do? Basically, if something has both a PositionComponent and VelocityComponent, we want the PositionComponent to update based on the VelocityComponent every frame.

It turns out Encompass provides a structure for this pattern, called a Detector. Let's use it now.

 In **game/engines/velocity.ts**:

```ts
import { Detector, Emits, Entity } from "encompass-ecs";
import { PositionComponent } from "game/components/position";
import { VelocityComponent } from "game/components/velocity";
import { MotionMessage } from "game/messages/component/motion";

@Emits(MotionMessage)
export class VelocityEngine extends Detector {
    public component_types = [ PositionComponent, VelocityComponent ];

    protected detect(entity: Entity) {
        const position_component = entity.get_component(PositionComponent);
        const velocity_component = entity.get_component(VelocityComponent);

        const motion_message = this.emit_component_message(MotionMessage, position_component);
        motion_message.x = velocity_component.x;
        motion_message.y = velocity_component.y;
    }
}
```

A Detector, like a Spawner, is an engine with one required property and method: *component_types* and *detect*.

When an Entity has all of the components specified in *component_types*, it begins to track the Entity. Each frame, it calls its *detect* method on that Entity.

So, our VelocityEngine will track everything with a PositionComponent and VelocityComponent and create a MotionMessage every frame.

Let's add our new Engine to the WorldBuilder:

```ts
world_builder.add_engine(VelocityEngine);
```

And add our new VelocityComponent in the BallSpawner.

```ts
const velocity_component = ball_entity.add_component(VelocityComponent);
velocity_component.x = 50;
velocity_component.y = -50;
```

Actually lets get rid of that magic value by adding velocity to the BallSpawnMessage.

**game/messages/ball_spawn.ts**

```ts
import { Message } from "encompass-ecs";

export class BallSpawnMessage extends Message {
    public x: number;
    public y: number;
    public size: number;
    public x_velocity: number;
    public y_velocity: number;
}
```

**game/engines/spawners/ball.ts**

```ts
const velocity_component = ball_entity.add_component(VelocityComponent);
velocity_component.x = message.x_velocity;
velocity_component.y = message.y_velocity;
```

**game/game.ts**

```ts
ball_spawn_message.x_velocity = 50;
ball_spawn_message.y_velocity = -50;
```

Let's run the game again.

<video width="75%" autoplay="autoplay" muted="muted" loop="loop" style="display: block; margin: 0 auto;">
    <source src="/images/moving_ball.webm" type="video/webm">
</video>

Still pretty boring but we're getting somewhere.
