---
title: "Collision Dispatch"
date: 2019-05-28T19:06:03-07:00
weight: 700
---

Let's make the CollisionDispatchEngine. All it needs to do is read the CollisionMessages and create specific collision messages from them.

In **games/engines/collision_dispatch.ts**:

```ts
import { Emits, Engine, Reads } from "encompass-ecs";
import { CollisionType } from "game/components/collision_types";
import { CollisionMessage } from "game/messages/collision";
import { BallPaddleCollisionMessage } from "game/messages/collisions/ball_paddle";
import { BallWallCollisionMessage } from "game/messages/collisions/ball_wall";
import { PaddleWallCollisionMessage } from "game/messages/collisions/paddle_wall";

@Reads(CollisionMessage)
@Emits(BallPaddleCollisionMessage, BallWallCollisionMessage, PaddleWallCollisionMessage)
export class CollisionDispatchEngine extends Engine {
    public update() {
        const collision_messages = this.read_messages(CollisionMessage);

        for (const collision_message of collision_messages.values()) {
            switch (collision_message.collision_type_one) {
                case CollisionType.ball:
                    switch (collision_message.collision_type_two) {
                        case CollisionType.paddle: {
                            // an exercise for the reader ;)
                        }

                        case CollisionType.wall: {
                            const message = this.emit_message(BallWallCollisionMessage);
                            message.ball_entity = collision_message.entity_one;
                            message.wall_entity = collision_message.entity_two;
                            message.ball_new_x = collision_message.entity_one_new_x;
                            message.ball_new_y = collision_message.entity_one_new_y;
                            message.normal = collision_message.collision_data.normal;
                            message.touch = collision_message.collision_data.touch;
                            break;
                        }
                    }
                    break;

                case CollisionType.paddle: {
                    switch (collision_message.collision_type_two) {
                        case CollisionType.wall: {
                            // another exercise for the reader ;)
                        }
                    }
                }
            }
        }
    }
}
```

Now we are emitting a BallWallCollisionMessage every time a ball collides with a wall. Why don't you try filling in the other collision messages yourself?

Don't forget to add it in **game.ts**

```ts
    world_builder.add_engine(CollisionDispatchEngine);
```

{{% notice notice %}}
Clever readers have probably noticed that this is a bit of an awkward structure. For our game, we only have three types of colliding entities we care about, so some switch statements work fine. What about a game with 20 different kinds of colliding entities? 100? We'd probably want a much more generic structure or this Engine's complexity would get out of hand.

What you really want to do, fundamentally, is map two collision types, independent of order, to a message emitting function. You'll probably need to implement a custom data structure to do this cleanly. It's very much outside of the scope of this tutorial for me to do this, but I wish you luck!
{{% /notice %}}
