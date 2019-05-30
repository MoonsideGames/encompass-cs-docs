---
title: "The Opponent"
date: 2019-05-30T13:11:35-07:00
weight: 500
---

Now that we have the ball moving around and bouncing, let's get the computer-controlled paddle working. We already have a lot of tools available to us from our implementation of the player paddle, so let's re-use as much as we can.

The computer-controlled paddle is essentially the same as the player-controlled paddle. The only difference is what causes it to move. So here's what we need to do.

- Designate in the PaddleSpawnMessage what is controlling the paddle
- Implement an engine that sends PaddleMoveMessages to the paddle

Let's revise our PaddleSpawnMessage. An enum type for our paddle control seems appropriate here.

```ts
import { Message } from "encompass-ecs";

export enum PaddleControlType {
    player_one,
    computer,
}

export class PaddleSpawnMessage extends Message {
    public x: number;
    public y: number;
    public width: number;
    public height: number;
    public move_speed: number;
    public control_type: PaddleControlType;
}
```

Then in the PaddleSpawner, where we add the PlayerOneComponent, let's put this instead:

```ts
    if (message.control_type === PaddleControlType.player_one) {
        paddle_entity.add_component(PlayerOneComponent);
    } else if (message.control_type === PaddleControlType.computer) {
        paddle_entity.add_component(PlayerComputerComponent);
    }
```

Now we can move on to the actual computer control behavior. The behavior of the Pong computer is pretty simple - it just moves the paddle towards the _y_ position of the ball.

We don't actually have a way to get the ball from the game state yet, so let's make another marker component.

In **game/components/ball.ts**:

```ts
import { Component } from "encompass-ecs";

export class BallComponent extends Component {}

```

and in our BallSpawner:

```ts
    ball_entity.add_component(BallComponent);
```

Now let's make our ComputerControlEngine. It will read the state of the game and tell the paddle to move in the direction of the ball.

In **game/engines/computer_control.ts**:

```ts
import { Emits, Engine } from "encompass-ecs";
import { BallComponent } from "game/components/ball";
import { PlayerComputerComponent } from "game/components/player_computer";
import { PositionComponent } from "game/components/position";
import { PaddleMoveMessage } from "game/messages/component/paddle_move";

@Emits(PaddleMoveMessage)
export class ComputerControlEngine extends Engine {
    public update() {
        const computer_components = this.read_components(PlayerComputerComponent);

        const ball_component = this.read_component(BallComponent);
        if (!ball_component) { return; }

        const ball_entity = this.get_entity(ball_component.entity_id);
        if (!ball_entity) { return; }

        const ball_position = ball_entity.get_component(PositionComponent);

        for (const computer_component of computer_components.values()) {
            const computer_entity = this.get_entity(computer_component.entity_id);

            if (computer_entity) {
                const computer_position = computer_entity.get_component(PositionComponent);

                if (computer_position.y - ball_position.y > 40) {
                    const message = this.emit_component_message(PaddleMoveMessage, computer_component);
                    message.direction = -1;
                } else if (computer_position.y - ball_position.y < -40) {
                    const message = this.emit_component_message(PaddleMoveMessage, computer_component);
                    message.direction = 1;
                }
            }
        }
    }
}
```

Notice how we are being careful not to assume the ball actually exists in this code. Remember - it never hurts to check if an object actually exists! You might save yourself from a nasty crash.

Don't forget to add the new Engine in **game.ts**!

```ts
world_builder.add_engine(ComputerControlEngine);
```

<video width="640" height="360" autoplay="autoplay" muted="muted" loop="loop" style="display: block; margin: 0 auto; width: 640;">
    <source src="/images/computer.webm" type="video/webm">
</video>

If we were doing this in an object-oriented way, we would have had to inherit from the paddle or introduce another state to the paddle, thus forcing us to refactor or increase the complexity of the paddle object itself.

Notice how in our case we didn't really have to change any of our existing logic - all we had to do was create a new component and write a new engine for producing behavior from that component, while getting to retain all the behavior we got from the other paddle components. See how clean and de-coupled this is? This is the power of _composition_ over _inheritance_.
