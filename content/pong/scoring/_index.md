---
title: "Scoring"
date: 2019-05-30T16:08:54-07:00
weight: 1000
---

In Pong, your opponent scores a point by getting the ball behind your paddle.

There's a couple of things we need to sort out here: 1) tracking and displaying each player's score, and 2) reacting appropriately when a ball collides with a goal (side) boundary.

Let's start by creating a ScoreComponent.

```ts
import { Component } from "encompass-ecs";

export class ScoreComponent extends Component {
    public score: number;
    public player_one: boolean;
}
```

And let's have a new collision type.

```ts
export enum CollisionType {
    ball,
    goal,
    paddle,
    wall,
}
```

We'll need a new BallGoalCollisionMessage:

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

And a new BallGoalCollisionEngine:

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
            const score_component = this.make_mutable(message.goal_entity.get_component(ScoreComponent));
            score_component.score += 1;

            message.ball_entity.destroy();
            this.collision_world.remove(message.ball_entity);
        }
    }
}
```

I've decided we should just attach the ScoreComponent directly to the goal entity for simplicity.

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

        const score_component = entity.add_component(ScoreComponent);
        score_component.score = 0;
        score_component.player_one = message.player_one;

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
