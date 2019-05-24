---
title: "Decoupling"
date: 2019-05-23T16:37:26-07:00
weight: 30
---

There's more to decoupling than just objects not directly referencing each other.

Really, when we talk about decoupling, we are saying that we don't want the structure of the program having unnecessary direct connections.

This is a pretty abstract principle, but there is a nice illustration of it in our program as it exists right now.

Right now our InputEngine is sending Messages to our MotionMessage. Is that actually what we want?

What I am saying is... can you think of an example where something other than direct input might want to control the movement of a paddle?

What about AI? What about an "attract mode"?

There's also a lot of structure about the paddle entity being embedded into the logic of the InputEngine. That doesn't feel good to me either.

Let's put something in between them.

We have to ask ourselves: how is it that we want the game to react when we press the buttons? We want the correct paddle to move up or down. That seems like a good abstraction for a Message.

Create a file: **game/engines/paddle_move_up.ts**

```ts
import { ComponentMessage, Message } from "encompass-ecs";
import { PlayerComponent } from "game/components/player";

export class PaddleMoveUpMessage extends Message implements ComponentMessage {
    public component: Readonly<PlayerOneComponent>;
}
```

and a file: **game/messages/paddle_move_down.ts**

```ts
import { ComponentMessage, Message } from "encompass-ecs";
import { PlayerComponent } from "game/components/player";

export class PaddleMoveDownMessage extends Message implements ComponentMessage {
    public component: Readonly<PlayerOneComponent>;
}
```

Uh oh, something seems weird here too. Why PlayerOneComponent? Sounds like we need some abstraction.

Create a file: **game/components/player.ts**

```ts
import { Component } from "encompass-ecs";

export abstract class PlayerComponent extends Component {}
```

*abstract* means that the class cannot be used directly, but must be inherited.

Change **games/component/player_one.ts**:

```ts
import { PlayerComponent } from "./player";

export class PlayerOneComponent extends PlayerComponent {}
```

And create **games/component/player_two.ts**:

```ts
import { PlayerComponent } from "./player";

export class PlayerTwoComponent extends PlayerComponent {}
```

One more thing: Why do we have separate MoveUp and MoveDown messages? The only distinction is the direction, which can easily be represented by a number. Let's consolidate everything.

Delete our PaddleMoveUpMessage and PaddleMoveDownMessage.

Create a file: **game/messages/paddle_move.ts**

```ts
import { ComponentMessage, Message } from "encompass-ecs";
import { PlayerComponent } from "game/components/player";

export class PaddleMoveMessage extends Message implements ComponentMessage {
    public component: Readonly<PlayerComponent>;
    public direction: number;
}
```

Now we're ready to rumble!!

Create a file: **game/engines/paddle_movement.ts**

```ts
import { Emits, Engine, Reads } from "encompass-ecs";
import { PaddleMoveSpeedComponent } from "game/components/paddle_move_speed";
import { PositionComponent } from "game/components/position";
import { MotionMessage } from "game/messages/component/motion";
import { PaddleMoveMessage } from "game/messages/component/paddle_move";

@Reads(PaddleMoveMessage)
@Emits(MotionMessage)
export class PaddleMovementEngine extends Engine {
    public update() {
        for (const message of this.read_messages(PaddleMoveMessage).values()) {
            const player_component = message.component;
            const player_entity = this.get_entity(player_component.entity_id);

            if (player_entity) {
                const position_component = player_entity.get_component(PositionComponent);
                const move_speed_component = player_entity.get_component(PaddleMoveSpeedComponent);

                const motion_message = this.emit_component_message(MotionMessage, position_component);
                motion_message.x = 0;
                motion_message.y = message.direction * move_speed_component.y;
            }
        }
    }
}
```

Finally, we revise our InputEngine to send PaddleMoveMessages.

```ts
import { Emits, Engine } from "encompass-ecs";
import { PlayerOneComponent } from "game/components/player_one";
import { PaddleMoveMessage } from "game/messages/component/paddle_move";

@Emits(PaddleMoveMessage)
export class InputEngine extends Engine {
    public update() {
        const player_one_component = this.read_component(PlayerOneComponent);

        if (player_one_component) {
            const player_one_entity = this.get_entity(player_one_component.entity_id);

            if (player_one_entity) {
                if (love.keyboard.isDown("up")) {
                    const message = this.emit_component_message(PaddleMoveMessage, player_one_component);
                    message.direction = -1;
                } else if (love.keyboard.isDown("down")) {
                    const message = this.emit_component_message(PaddleMoveMessage, player_one_component);
                    message.direction = 1;
                }
            }
        }
    }
}
```

And don't forgot to add our new Engine to the WorldBuilder.

```ts
world_builder.add_engine(PaddleMovementEngine);
```

Look at how concise and easy to understand everything is now! And when we get around to adding Player 2 we'll barely have to do any work at all, because everything that drives Player 1 can be used to drive Player 2.

Decoupling isn't just a law we follow just because someone told us it was a good idea or whatever. It exists as a principle to remind us to try to express our ideas at a higher level, so we can write more powerful and concise code and create less room for error.

When you get more experienced you'll be able to sniff out tightly coupled code very easily. Don't stress about it too much, but keep the idea in the back of your mind always.

Before we move on, let's make the paddle a little zippier.

```ts
move_speed_component.y = 400;
```

<video width="640" height="360" autoplay="autoplay" muted="muted" loop="loop" style="display: block; margin: 0 auto; width: 640;">
    <source src="/images/better_paddle.webm" type="video/webm">
</video>

That's more like it.
