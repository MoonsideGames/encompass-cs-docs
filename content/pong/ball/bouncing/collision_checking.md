---
title: "Collision Checking"
date: 2019-05-28T18:51:15-07:00
weight: 600
---

In **game/engines/collision_message.ts**:

```ts
import { Entity, Message } from "encompass-ecs";
import { CollisionType } from "game/components/collision_types";
import { Collision } from "lua-lib/bump";

export class CollisionMessage extends Message {
    public entity_one: Entity;
    public entity_two: Entity;
    public collision_type_one: CollisionType;
    public collision_type_two: CollisionType;
    public entity_one_new_x: number;
    public entity_one_new_y: number;
    public entity_two_new_x: number;
    public entity_two_new_y: number;
    public collision_data: Collision;
}
```

Let's break down what we want collision detection to actually do.

First, we tell the Collision World about the current positions of the objects. Next we check each object for collisions by using the "check" method, which takes the proposed new position of the object and gives us collision information in return.

For every collision that we find, we create a CollisionMessage for it. Why are we comparing the collision types? Let's say we want to have a BallWallCollisionMessage. Obviously we will want to know which entity in the collision represents the ball and which one represents the wall. So we just sort them at this step for convenience.

In **game/engines/collision_check.ts**:

```ts
import { Emits, Engine, Entity, Reads } from "encompass-ecs";
import { BoundariesComponent } from "game/components/boundaries";
import { CollisionTypesComponent } from "game/components/collision_types";
import { PositionComponent } from "game/components/position";
import { CollisionMessage } from "game/messages/collision";
import { CollisionCheckMessage } from "game/messages/collision_check";
import { World } from "lua-lib/bump";

@Reads(CollisionCheckMessage)
@Emits(CollisionMessage)
export class CollisionCheckEngine extends Engine {
    private collision_world: World;

    public initialize(collision_world: World) {
        this.collision_world = collision_world;
    }

    public update() {
        const collision_check_messages = this.read_messages(CollisionCheckMessage);

        // update all positions in collision world
        for (const message of collision_check_messages.values()) {
            const entity = message.entity;
            const position = entity.get_component(PositionComponent);
            const boundaries = entity.get_component(BoundariesComponent);

            this.collision_world.update(
                message.entity,
                position.x - boundaries.width * 0.5,
                position.y - boundaries.height * 0.5
            );
        }

        // perform collision checks with new positions
        for (const message of collision_check_messages.values()) {
            const entity = message.entity;
            const position = entity.get_component(PositionComponent);
            const boundaries = entity.get_component(BoundariesComponent);
            const x = position.x + message.x_delta;
            const y = position.y + message.y_delta;

            const [new_x, new_y, cols, len] = this.collision_world.check(
                entity,
                x - boundaries.width * 0.5,
                y - boundaries.height * 0.5,
                () => "touch"
            );

            for (const col of cols) {
                const other = col.other as Entity;
                const other_position = other.get_component(PositionComponent);

                for (const collision_type_one of entity.get_component(CollisionTypesComponent)!.collision_types) {
                    for (const collision_type_two of other.get_component(CollisionTypesComponent)!.collision_types) {
                        const collision_message = this.emit_message(CollisionMessage);
                        if (collision_type_one < collision_type_two) {
                            collision_message.entity_one = entity;
                            collision_message.entity_two = other;
                            collision_message.collision_type_one = collision_type_one;
                            collision_message.collision_type_two = collision_type_two;
                            collision_message.entity_one_new_x = x;
                            collision_message.entity_one_new_y = y;
                            collision_message.entity_two_new_x = other_position.x;
                            collision_message.entity_two_new_y = other_position.y;
                            collision_message.collision_data = col;
                        } else {
                            collision_message.entity_one = other;
                            collision_message.entity_two = entity;
                            collision_message.collision_type_one = collision_type_two;
                            collision_message.collision_type_two = collision_type_one;
                            collision_message.entity_one_new_x = other_position.x;
                            collision_message.entity_one_new_y = other_position.y;
                            collision_message.entity_two_new_x = x;
                            collision_message.entity_two_new_y = y;
                            collision_message.collision_data = col;
                        }
                    }
                }
            }
        }
    }
}
```

The "initialize" method gives the Engine a reference to the Collision World that needs to be shared by everything that deals with collision. Let's make sure to call the "initialize" method from **game.ts**:

```ts
        const collision_world = CollisionWorld.newWorld(32);

        ...

        world_builder.add_engine(CollisionCheckEngine).initialize(collision_world);
```
