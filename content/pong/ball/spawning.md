---
title: "Spawning"
date: 2019-05-23T18:09:07-07:00
weight: 10
---

In **game/game.ts**...

```ts
const ball_entity = world_builder.create_entity();

const ball_position_component = ball_entity.add_component(PositionComponent);
ball_position_component.x = 640;
ball_position_component.y = 360;

const ball_size = 16;
const ball_canvas = love.graphics.newCanvas(ball_size, ball_size);
love.graphics.setCanvas(ball_canvas);
love.graphics.setBlendMode("alpha");
love.graphics.setColor(1, 1, 1, 1);
love.graphics.rectangle("fill", 0, 0, ball_size, ball_size);
love.graphics.setCanvas();

const ball_canvas_component = ball_entity.add_component(CanvasComponent);
ball_canvas_component.canvas = ball_canvas;
ball_canvas_component.x_scale = 1;
ball_canvas_component.y_scale = 1;
```

OK. We're both thinking it. Why is all this crap going straight in **game.ts**? And there's magic values everywhere! You are absolutely right. Encompass actually has a built-in abstraction Engine for creating new Entities called a Spawner. Let's use one.

Let's create a new folder: **game/engines/spawners**

And a new file: **game/engines/spawners/ball.ts**

```ts
import { Reads, Spawner } from "encompass-ecs";
import { CanvasComponent } from "game/components/canvas";
import { PositionComponent } from "game/components/position";
import { BallSpawnMessage } from "game/messages/ball_spawn";

@Reads(BallSpawnMessage)
export class BallSpawner extends Spawner {
    public spawn_message_type = BallSpawnMessage;

    public spawn(message: BallSpawnMessage) {
        const ball_entity = this.create_entity();

        const ball_position_component = ball_entity.add_component(PositionComponent);
        ball_position_component.x = message.x;
        ball_position_component.y = message.y;

        const ball_canvas = love.graphics.newCanvas(message.size, message.size);
        love.graphics.setCanvas(ball_canvas);
        love.graphics.setBlendMode("alpha");
        love.graphics.setColor(1, 1, 1, 1);
        love.graphics.rectangle("fill", 0, 0, message.size, message.size);
        love.graphics.setCanvas();

        const ball_canvas_component = ball_entity.add_component(CanvasComponent);
        ball_canvas_component.canvas = ball_canvas;
        ball_canvas_component.x_scale = 1;
        ball_canvas_component.y_scale = 1;
    }
}
```

Spawners aren't very complicated. They have one required property and method: *spawn_message_type* and *spawn*.

When the Spawner reads a message of *spawn_message_type*, it runs its spawn method once. Simple as that.

Now let's actually send out the BallSpawnMessage.

In **game/game.ts**...

```ts
...

    world_builder.add_engine(BallSpawner);

...

    const ball_spawn_message = world_builder.emit_message(BallSpawnMessage);
    ball_spawn_message.x = 640;
    ball_spawn_message.y = 360;
    ball_spawn_message.size = 16;

...
```

![boring ball](/images/paddle_with_ball.png)

Well, it draws, but it's a bit boring without any movement. Let's make it move around.
