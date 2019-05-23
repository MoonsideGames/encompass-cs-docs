---
title: "Motion Engine"
date: 2019-05-23T13:03:39-07:00
weight: 5
---

To create an Engine, we extend the Engine class.

Create a file: **game/engines/motion.ts**

```ts
import { Engine } from "encompass-ecs";

export class MotionEngine extends Engine {
    public update(dt: number) {}
}
```

Every Engine needs an *update* method, which optionally takes a *delta-time* value as a parameter.

*delta-time* is simply the time that has elapsed between the last frame and the current one in seconds. We'll talk more about why this is important in a minute.

Let's think for a minute about what we want this Engine to actually *do*. Motion is just the change of position over time, right? So our MotionEngine is going to modify PositionComponents based on some amount of movement.

We're gonna need a Message. More specifically, a ComponentMessage.

Create a file: **game/messages/component/motion.ts**

```ts
import { ComponentMessage, Message } from "encompass-ecs";
import { PositionComponent } from "game/components/position";

export class MotionMessage extends Message implements ComponentMessage {
    public component: Readonly<PositionComponent>;
    public x: number;
    public y: number;
}
```

*implements* means that the class defines certain required properties or methods. If you don't understand it right now, don't worry, just know that in this case, a Message that *implements* ComponentMessage needs to have a *component* property. In our case, a MotionMessage wants to refer to some specific PositionComponent that needs to be updated.

{{% notice warning %}}
Why is the component type wrapped in *Readonly*? You can actually get away with not doing this, but it means you can accidentally get around some of the safety features of Encompass that prevent race conditions. So make sure you do this when defining a ComponentMessage.
{{% /notice %}}

{{% notice tip %}}
Remember before when I said that it is a big no-no to have Components reference each other? Well, it's perfectly fine to have Messages refer to a Component, or even multiple Components.

**Don't** ever have a Message that refers to another Message though. That is very bad.
{{% /notice %}}

Now, how is our MotionEngine going to interact with MotionMessages? It's going to Read them.

```ts
import { Engine, Reads } from "encompass-ecs";
import { MotionMessage } from "game/messages/component/motion";

@Reads(MotionMessage)
export class MotionEngine extends Engine {
    public update(dt: number) {
        const motion_messages = this.read_messages(MotionMessage);
    }
}
```

What happens if we don't declare **@Reads** but still call *read_messages*? Encompass will yell at us when the game runs, because then it can't guarantee that this Engine runs after Engines which *emit* MotionMessages, which is no good. We'll talk about Emitting messages soon.

Now we have a reference to all MotionMessages that were emitted this frame. Let's use them to update PositionComponents.

```ts
import { Engine, Reads } from "encompass-ecs";
import { MotionMessage } from "game/messages/component/motion";

@Reads(MotionMessage)
export class MotionEngine extends Engine {
    public update(dt: number) {
        const motion_messages = this.read_messages(MotionMessage);
        for (const message of motion_messages.values()) {
            const position_component = message.component;
            position_component.x += message.x;
            position_component.y += message.y;
        }
    }
}
```

Uh oh. The compiler is yelling at us. "Cannot assign to 'x' because it is a read-only property." We're going to need to make the component Mutable.

Mutable is a scary word, but it really just means "can have its properties changed." We *really* don't want two different Engines to be able to change the same Component type, because then we can't be certain about what the final result of the changes will be, and that is an opportunity for horrible nasty bugs to lurk in our game.

So if we're going to be changing PositionComponents, the Engine needs to declare that it Mutates them, and then make the Component mutable.

```ts
import { Engine, Mutates, Reads } from "encompass-ecs";
import { PositionComponent } from "game/components/position";
import { MotionMessage } from "game/messages/component/motion";

@Reads(MotionMessage)
@Mutates(PositionComponent)
export class MotionEngine extends Engine {
    public update(dt: number) {
        const motion_messages = this.read_messages(MotionMessage);
        for (const message of motion_messages.values()) {
            const position_component = this.make_mutable(message.component);
            position_component.x += message.x;
            position_component.y += message.y;
        }
    }
}
```

Now the compiler is content, and so are we.

Let's add this Engine to our WorldBuilder before we forget.

In **game/game.ts**

```ts
...

    public load() {
        this.canvas = love.graphics.newCanvas();

        const world_builder = new WorldBuilder();

        // ADD YOUR ENGINES HERE...
        world_builder.add_engine(MotionEngine);

        ...

    }
```

Of course, if we run the game now, nothing will happen, because nothing is actually sending out MotionMessages. Let's make that happen.
