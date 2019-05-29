---
title: "Motion Engine: The Revenge"
date: 2019-05-28T18:01:49-07:00
weight: 500
---

Let's rewrite our MotionEngine.

In **game/messages/collision_check.ts**:

```ts
import { Entity, Message } from "encompass-ecs";

export class CollisionCheckMessage extends Message {
    public entity: Entity;
    public x_delta: number;
    public y_delta: number;
}
```

In **game/messages/update_position.ts**:

```ts
import { ComponentMessage, Message } from "encompass-ecs";
import { PositionComponent } from "game/components/position";

export class UpdatePositionMessage extends Message implements ComponentMessage {
    public component: Readonly<PositionComponent>;
    public x_delta: number;
    public y_delta: number;
}
```

Here's the process we'll follow for our MotionEngine:

We associate MotionMessages with their PositionComponents. We consolidate them to get a total "x_delta" and a "y_delta". We create an UpdatePositionMessage containing these values. Next, we create CollisionCheckMessages containing the delta values if the PositionComponent's entity has a BoundingBoxComponent.

Finally, we go over all BoundingBoxComponents that didn't have MotionMessages associated with them and create CollisionCheckMessages for those too. Otherwise things that didn't move wouldn't be collision checked, and that would not be correct.

```ts
import { Emits, Engine, Reads } from "encompass-ecs";
import { BoundingBoxComponent } from "game/components/bounding_box";
import { PositionComponent } from "game/components/position";
import { CollisionCheckMessage } from "game/messages/collision_check";
import { MotionMessage } from "game/messages/component/motion";
import { UpdatePositionMessage } from "game/messages/update_position";
import { GCOptimizedList, GCOptimizedSet } from "tstl-gc-optimized-collections";

@Reads(MotionMessage)
@Emits(UpdatePositionMessage, CollisionCheckMessage)
export class MotionEngine extends Engine {
    private component_to_message = new Map<PositionComponent, GCOptimizedList<MotionMessage>>();
    private bounding_box_set = new GCOptimizedSet<BoundingBoxComponent>();

    public update(dt: number) {
        const motion_messages = this.read_messages(MotionMessage);
        for (const message of motion_messages.values()) {
            this.register_message(message);
        }

        for (const [position_component, messages] of this.component_to_message.entries()) {
            const entity = this.get_entity(position_component.entity_id)!;

            let x_delta = 0;
            let y_delta = 0;

            for (const message of messages.values()) {
                x_delta += message.x * dt;
                y_delta += message.y * dt;
            }

            const update_position_message = this.emit_component_message(UpdatePositionMessage, position_component);
            update_position_message.x_delta = x_delta;
            update_position_message.y_delta = y_delta;

            if (entity.has_component(BoundingBoxComponent)) {
                const collision_check_message = this.emit_message(CollisionCheckMessage);
                collision_check_message.entity = entity;
                collision_check_message.x_delta = x_delta;
                collision_check_message.y_delta = y_delta;

                this.bounding_box_set.add(entity.get_component(BoundingBoxComponent));
            }
        }

        for (const component of this.read_components(BoundingBoxComponent).values()) {
            if (!this.bounding_box_set.has(component)) {
                const collision_check_message = this.emit_message(CollisionCheckMessage);
                collision_check_message.entity = this.get_entity(component.entity_id)!;
                collision_check_message.x_delta = 0;
                collision_check_message.y_delta = 0;
            }
        }

        this.component_to_message.clear();
    }

    private register_message(message: MotionMessage) {
        if (!this.component_to_message.has(message.component)) {
            this.component_to_message.set(message.component, new GCOptimizedList<MotionMessage>());
        }
        this.component_to_message.get(message.component)!.add(message);
    }
}
```

Now let's detect collisions.
