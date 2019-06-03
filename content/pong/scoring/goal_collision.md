---
title: "Goal Collision"
date: 2019-06-03T12:39:11-07:00
weight: 10
---

First we will need a new collision type for goals.

```ts
export enum CollisionType {
    ball,
    goal,
    paddle,
    wall,
}
```

And a BallGoalCollisionMessage:

```ts
import { Entity, Message } from "encompass-ecs";

export class BallGoalCollisionMessage extends Message {
    public ball_entity: Entity;
    public goal_entity: Entity;
}
```

And some dispatch logic in the CollisionDispatchEngine:

```ts
...

switch (collision_message.collision_type_one) {
    case CollisionType.ball:
        switch (collision_message.collision_type_two) {
            case CollisionType.goal: {
                const message = this.emit_message(BallGoalCollisionMessage);
                message.ball_entity = collision_message.entity_one;
                message.goal_entity = collision_message.entity_two;
                break;
            }

...
```

Don't forget to declare the BallGoalCollisionMessage in @Emits!

Let's create our BallGoalCollisionEngine:

```ts
import { Engine, Mutates, Reads } from "encompass-ecs";
import { ScoreComponent } from "game/components/score";
import { BallGoalCollisionMessage } from "game/messages/collisions/ball_goal";
import { World } from "lua-lib/bump";

@Reads(BallGoalCollisionMessage)
@Mutates(ScoreComponent)
export class BallGoalCollisionEngine extends Engine {
    private collision_world: World;

    public initialize(collision_world: World) {
        this.collision_world = collision_world;
    }

    public update() {
        for (const message of this.read_messages(BallGoalCollisionMessage).values()) {
            message.ball_entity.destroy();
            this.collision_world.remove(message.ball_entity);
        }
    }
}
```

When we detect a collision between a ball and a goal, we destroy the ball entity and, importantly, remember to remove its rectangle from the collision world.

{{% notice tip %}}
It is generally much more performance-efficient to deactivate entities instead of destroying and respawn them, but it's slightly more complicated to do so. In our case, it will be fine to destroy and respawn.
{{% /notice %}}

We'll need a way to create our goal entities now.

Let's have a new GoalSpawnMessage:

```ts
import { Message } from "encompass-ecs";

export class GoalSpawnMessage extends Message {
    public x: number;
    public y: number;
    public width: number;
    public height: number;
    public player_one: boolean;
}
```

And a new GoalSpawner:

```ts
import { Spawner } from "encompass-ecs";
import { BoundingBoxComponent } from "game/components/bounding_box";
import { CollisionType, CollisionTypesComponent } from "game/components/collision_types";
import { PositionComponent } from "game/components/position";
import { ScoreComponent } from "game/components/score";
import { GoalSpawnMessage } from "game/messages/goal_spawn";
import { World } from "lua-lib/bump";

export class GoalSpawner extends Spawner {
    public spawn_message_type = GoalSpawnMessage;

    private collision_world: World;

    public initialize(collision_world: World) {
        this.collision_world = collision_world;
    }

    public spawn(message: GoalSpawnMessage) {
        const entity = this.create_entity();

        const boundaries = entity.add_component(BoundingBoxComponent);
        boundaries.width = message.width;
        boundaries.height = message.height;

        const position = entity.add_component(PositionComponent);
        position.x = message.x;
        position.y = message.y;

        const collision_types_component = entity.add_component(CollisionTypesComponent);
        collision_types_component.collision_types = [ CollisionType.goal ];

        this.collision_world.add(
            entity,
            message.x - message.width * 0.5,
            message.y - message.height * 0.5,
            message.width,
            message.height
        );
    }
}
```

Notice this is very similar to our GameBoundarySpawner. I'm not thrilled about that, so we could maybe consolidate it later. Let's leave it for now because we're not quite done with this feature.

Now let's work on a mechanism for respawning the ball.
