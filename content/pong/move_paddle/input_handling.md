---
title: "Input Handling"
date: 2019-05-23T13:38:42-07:00
weight: 10
---

In Pong, the paddles move when the player moves the joystick on their controller up or down.

We currently have a MotionEngine that reads MotionMessages and moves the PositionComponents they reference.

So... it makes sense that we would have an InputEngine that sends MotionMessages, yeah?

Create a file: **game/engines/input.ts**

```ts
import { Engine } from "encompass-ecs";
import { MotionMessage } from "game/messages/component/motion";

export class InputEngine extends Engine {
    public update() {
        if (love.keyboard.isDown("up")) {
            this.emit_component_message(MotionMessage,
        }
    }
}
```

*record scratch*

Uh oh. *Engine.emit_component_message* emits a Component Message, as the name suggests. But it needs an actual component to attach to the message. How do we give the message a reference to our paddle entity's position?

Sounds like we need another Component.

One thing we can use Components for is a concept I call *marking* or *tagging*. Essentially, we use a Component to designate that an Entity is a certain kind of object in the game.

Create a file: **component/player_one.ts**

```ts
import { Component } from "encompass-ecs";

export class PlayerOneComponent extends Component {}
```

That's it! The component itself doesn't need any information on it. Its mere existence on the Entity will tell us that this Entity represents Player 1.

Let's add it to our paddle Entity.

In **game/game.ts**:

```ts
...

paddle_entity.add_component(PlayerOneComponent);

...
```

Now we can go back to our InputEngine.

```ts
import { Emits, Engine } from "encompass-ecs";
import { PlayerOneComponent } from "game/components/player_one";
import { PositionComponent } from "game/components/position";
import { MotionMessage } from "game/messages/component/motion";

@Emits(MotionMessage)
export class InputEngine extends Engine {
    public update() {
        const player_one_component = this.read_component(PlayerOneComponent);

        if (player_one_component) {
            const player_one_entity = this.get_entity(player_one_component.entity_id);

            if (player_one_entity) {
                const player_one_position_component = player_one_entity.get_component(PositionComponent);

                if (love.keyboard.isDown("up")) {
                    const message = this.emit_component_message(MotionMessage, player_one_position_component);
                    message.x = 0;
                    message.y = -10;
                }
            }
        }
    }
}
```

Ok... what the heck is *this.read_component*?

Engines have total freedom to read anything in the game state that they desire. This gives Engines a tremendous amount of flexibility to do what they need to do.

In this case, we are reading the game state to find our PlayerOneComponent. From there, we can get the Entity to which the PlayerOneComponent belongs. Then we can get the PositionComponent of that Entity, and send a message about it when the "up" key is pressed down.

{{% notice warning %}}
Similar to *Entity.get_component* and *Entity.get_components*, Engines have *Engine.read_component* and *Engine.read_components*. If you try to do the singular *read_component* on a game state that has multiple components of that type, an error will be thrown. So be careful that your singleton components are actually singletons!
{{% /notice %}}

Also, remember when we had to declare **@Reads** on our MotionEngine? Well, similarly, we have to declare **@Emits** when our Engine emits a certain kind of Message. Otherwise Encompass will get mad at us and crash the game for our own safety.

Let's add our InputEngine to the WorldBuilder.

In **game/game.ts**:

```ts
// ADD YOUR ENGINES HERE...
world_builder.add_engine(InputEngine);
world_builder.add_engine(MotionEngine);
```

It doesn't matter which order they go in, because remember, Encompass figures it out automatically. I just prefer this order for some reason. Once we have a lot of Engines it stops mattering pretty quickly anyway.

Let's run the game again!!
