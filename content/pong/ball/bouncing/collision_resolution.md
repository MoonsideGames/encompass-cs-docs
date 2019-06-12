---
title: "Collision Resolution"
date: 2019-05-28T20:39:54-07:00
weight: 800
---

What do we want to actually happen when a ball collides with a wall?

Obviously the wall doesn't do anything. It just sits there. That's easy!

The ball needs to bounce off of the wall. We can calculate exactly where it should end up by adding distance along the collision normal equal to twice the difference between the proposed location of the ball and where it touched the wall.

{{% notice tip %}}
**What the heck is a collision normal?**

You can think of the collision normal as just an arrow pointing away from the wall. If you want more details about this, check out the [bump.lua README](https://github.com/kikito/bump.lua/blob/master/README.md). It has illustrations of collisions and the normal vectors they create.
{{% /notice %}}

In **game/engines/collision/ball_wall.ts**:

```ts
import { Emits, Engine, Reads } from "encompass-ecs";
import { BoundingBoxComponent } from "game/components/bounding_box";
import { PositionComponent } from "game/components/position";
import { VelocityComponent } from "game/components/velocity";
import { BallWallCollisionMessage } from "game/messages/collisions/ball_wall";
import { UpdatePositionMessage } from "game/messages/update_position";
import { UpdateVelocityMessage } from "game/messages/update_velocity";

@Reads(BallWallCollisionMessage)
@Emits(UpdatePositionMessage, UpdateVelocityMessage)
export class BallWallCollisionEngine extends Engine {
    public update() {
        for (const message of this.read_messages(BallWallCollisionMessage).values()) {
            const ball_position = message.ball_entity.get_component(PositionComponent);
            const ball_velocity = message.ball_entity.get_component(VelocityComponent);
            const ball_bounding_box = message.ball_entity.get_component(BoundingBoxComponent);

            const velocity_message = this.emit_component_message(UpdateVelocityMessage, ball_velocity);
            velocity_message.x_delta = 2 * message.normal.x * Math.abs(ball_velocity.x);
            velocity_message.y_delta = 2 * message.normal.y * Math.abs(ball_velocity.y);

            // calculate bounce, remembering to re-transform coordinates to origin space
            const y_distance = Math.abs(message.ball_new_y - (message.touch.y + ball_bounding_box.height * 0.5));
            const x_distance = Math.abs(message.ball_new_x - (message.touch.x + ball_bounding_box.width * 0.5));

            const position_message = this.emit_component_message(UpdatePositionMessage, ball_position);
            position_message.x_delta = 2 * message.normal.x * x_distance;
            position_message.y_delta = 2 * message.normal.y * y_distance;
        }
    }
}
```

Notice that we also want to update the velocity when the ball bounces. Let's create that UpdateVelocity behavior.

In **game/messages/update_velocity.ts**:

```ts
import { ComponentMessage, Message } from "encompass-ecs";
import { VelocityComponent } from "game/components/velocity";

export class UpdateVelocityMessage extends Message implements ComponentMessage {
    public component: Readonly<VelocityComponent>;
    public x_delta: number;
    public y_delta: number;
}
```

In **game/engines/update_velocity.ts**:

```ts
import { ComponentModifier, Mutates, Reads } from "encompass-ecs";
import { VelocityComponent } from "game/components/velocity";
import { UpdateVelocityMessage } from "game/messages/update_velocity";
import { GCOptimizedSet } from "encompass-gc-optimized-collections";

@Reads(UpdateVelocityMessage)
@Mutates(VelocityComponent)
export class UpdateVelocityEngine extends ComponentModifier {
    public component_message_type = UpdateVelocityMessage;

    public modify(component: VelocityComponent, messages: GCOptimizedSet<UpdateVelocityMessage>) {
        for (const message of messages.entries()) {
            component.x += message.x_delta;
            component.y += message.y_delta;
        }
    }
}
```

Our BallPaddleCollisionEngine will behave the exact same way. Why don't you try to fill it in yourself?

Finally, we want to make sure our paddles don't go past the game boundary.

```ts
import { Emits, Engine, Reads } from "encompass-ecs";
import { BoundingBoxComponent } from "game/components/bounding_box";
import { PositionComponent } from "game/components/position";
import { PaddleWallCollisionMessage } from "game/messages/collisions/paddle_wall";
import { UpdatePositionMessage } from "game/messages/update_position";

@Reads(PaddleWallCollisionMessage)
@Emits(UpdatePositionMessage)
export class PaddleWallCollisionEngine extends Engine {
    public update() {
        for (const message of this.read_messages(PaddleWallCollisionMessage).values()) {
            const paddle_position = message.paddle_entity.get_component(PositionComponent);
            const paddle_bounding_box = message.paddle_entity.get_component(BoundingBoxComponent);

            const x_distance = Math.abs(message.paddle_new_x - (message.touch.x + paddle_bounding_box.width * 0.5));
            const y_distance = Math.abs(message.paddle_new_y - (message.touch.y + paddle_bounding_box.height * 0.5));

            const position_message = this.emit_component_message(UpdatePositionMessage, paddle_position);
            position_message.x_delta = message.normal.x * x_distance;
            position_message.y_delta = message.normal.y * y_distance;
        }
    }
}
```

That's it for defining our collision behavior!
