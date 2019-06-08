---
title: "Paddle Bounce"
date: 2019-06-08T14:46:08-07:00
weight: 10
---

One thing that isn't quite right in our game is that when the ball bounces, it reflects directly off the paddle.

If we look at the original Pong, we see that the angle of the ball is actually affected by the position where the ball hits the paddle. If the ball hits the edges of the paddle, it bounces off at a wider angle. If it hits the center of the paddle, it bounces horizontally.

This is a classic risk/reward mechanic: it's safer to hit the center of the paddle, but it's easier for your opponent to return the shot. It's riskier to hit the edges of the paddle, but the shot is more difficult to return.

Let's return to our BallPaddleCollisionEngine.

```ts
const velocity_message = this.emit_component_message(UpdateVelocityMessage, ball_velocity);
velocity_message.x_delta = 2 * message.normal.x * Math.abs(ball_velocity.x);
velocity_message.y_delta = 2 * message.normal.y * Math.abs(ball_velocity.y);
```

This is what is reflecting our ball's velocity.

Let's calculate a new velocity based on where the ball touches the paddle.

First, we don't want the speed of the ball to change. So let's store it. The way we calculate speed is by obtaining the length of the velocity. We also want to know if the ball was travelling right or left at the time of contact.

```ts
const speed = len(ball_velocity.x, ball_velocity.y);
const horizontal = ball_velocity.x < 0 ? 1 : -1;
```

Now we want to calculate our new angle. First, we want to check how far the point of contact was from the center of the paddle. The problem is that we don't know inside our engine how big the paddle is. Let's fix that.

Let's return to our PaddleMoveSpeedComponent.

```ts
import { Component } from "encompass-ecs";

export class PaddleMoveSpeedComponent extends Component {
    public y: number;
}
```

I think we should just store all of our paddle-related information here, so let's rename it to PaddleComponent, and rename _y_ to *move_speed*. Make sure to use VSCode's rename feature so you don't have to manually tweak the names in multiple files.

```ts
import { Component } from "encompass-ecs";

export class PaddleComponent extends Component {
    public height: number;
    public move_speed: number;
}
```

Let's change our PaddleSpawner to comply with our changed component.

```ts
const paddle_component = paddle_entity.add_component(PaddleComponent);
paddle_component.move_speed = message.move_speed;
paddle_component.height = height;
```

Now, back in our collision engine, we can get a new rotation. First, we figure out how far the contact was from the center of the paddle. Then we convert that to a number ranging from -1 to 1. Finally, we multiply that number by pi/4 to get a number ranging from -pi/4 to pi/4.

```ts
const diff = message.touch.y - paddle_y;
const scale = diff / (paddle_height * 0.5);
const rotation = (scale * math.pi / 4);
```

Then we perform the rotation on a vector pointing directly to the right, with the same length we calculated earlier. We multiply it to reverse its direction if it was originally travelling to the right.

```ts
let [new_x_velocity, new_y_velocity] = rotate(rotation, speed, 0);
new_x_velocity *= horizontal;
```

Finally, we create our velocity deltas by subtracting our old velocity from our new desired velocity.

```ts
[
    velocity_message.x_delta,
    velocity_message.y_delta,
] = sub(new_x_velocity, new_y_velocity, ball_velocity.x, ball_velocity.y);
```

Our final result looks like this:

```ts
import { Emits, Engine, Reads } from "encompass-ecs";
import { BoundingBoxComponent } from "game/components/bounding_box";
import { PaddleComponent } from "game/components/paddle";
import { PositionComponent } from "game/components/position";
import { VelocityComponent } from "game/components/velocity";
import { BallPaddleCollisionMessage } from "game/messages/collisions/ball_paddle";
import { UpdatePositionMessage } from "game/messages/update_position";
import { UpdateVelocityMessage } from "game/messages/update_velocity";
import { len, rotate, sub } from "lua-lib/hump/vectorlight";

@Reads(BallPaddleCollisionMessage)
@Emits(UpdatePositionMessage, UpdateVelocityMessage)
export class BallPaddleCollisionEngine extends Engine {
    public update() {
        for (const message of this.read_messages(BallPaddleCollisionMessage).values()) {
            const ball_position = message.ball_entity.get_component(PositionComponent);
            const ball_velocity = message.ball_entity.get_component(VelocityComponent);
            const ball_boundaries = message.ball_entity.get_component(BoundingBoxComponent);

            const paddle_height = message.paddle_entity.get_component(PaddleComponent).height;
            const paddle_y = message.paddle_entity.get_component(PositionComponent).y;

            const velocity_message = this.emit_component_message(UpdateVelocityMessage, ball_velocity);

            // calculate new ball velocity based on paddle contact
            const speed = len(ball_velocity.x, ball_velocity.y);
            const horizontal = ball_velocity.x < 0 ? 1 : -1;

            const diff = message.touch.y - paddle_y;
            const scale = diff / (paddle_height * 0.5);
            const rotation = (scale * math.pi / 4);

            let [new_x_velocity, new_y_velocity] = rotate(rotation, speed, 0);
            new_x_velocity *= horizontal;

            [
                velocity_message.x_delta,
                velocity_message.y_delta,
            ] = sub(new_x_velocity, new_y_velocity, ball_velocity.x, ball_velocity.y);

            // calculate bounce position, remembering to re-transform coordinates to origin space
            const y_distance = Math.abs(message.ball_new_y - (message.touch.y + ball_boundaries.height * 0.5));
            const x_distance = Math.abs(message.ball_new_x - (message.touch.x + ball_boundaries.width * 0.5));

            const position_message = this.emit_component_message(UpdatePositionMessage, ball_position);
            position_message.x_delta = 2 * message.normal.x * x_distance;
            position_message.y_delta = 2 * message.normal.y * y_distance;
        }
    }
}
```

<video width="75%" height="360" autoplay="autoplay" muted="muted" loop="loop" style="display: block; margin: 0 auto;">
    <source src="/images/bounce_angle.mp4" type="video/mp4">
</video>

Now we have a game on our hands!
