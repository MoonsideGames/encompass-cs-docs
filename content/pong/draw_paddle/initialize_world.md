---
title: "Initializing the World"
date: 2019-05-23T12:06:18-07:00
weight: 15
---

It's time to put it all together.

Let's look at our **game/game.ts** file. The *load* method looks like this:

```ts
public load() {
    this.canvas = love.graphics.newCanvas();

    const world_builder = new WorldBuilder();

    // ADD YOUR ENGINES HERE...

    // ADD YOUR RENDERERS HERE...

    // ADD YOUR STARTING ENTITIES HERE...

    this.world = world_builder.build();
}
```

Let's do as the helpful file asks, eh?

```ts
import { CanvasRenderer } from "./renderers/canvas";
...

export class Game {
    ...

    public load() {
        this.canvas = love.graphics.newCanvas();

        const world_builder = new WorldBuilder();

        // ADD YOUR ENGINES HERE...

        // ADD YOUR RENDERERS HERE...
        world_builder.add_renderer(CanvasRenderer);

        // ADD YOUR STARTING ENTITIES HERE...

        this.world = world_builder.build();
    }

    ...
```

Now our CanvasRenderer will exist in the world. We only have two things left to do: create a Canvas that contains our paddle visuals, and put it on an Entity.

Let's tell the World Builder that we want a new Entity. This will be our paddle Entity.

```ts
const paddle_entity = world_builder.create_entity();
```

Let's set up our paddle Canvas.

```ts
const width = 4;
const height = 8;

const paddle_canvas = love.graphics.newCanvas(4, 8);
love.graphics.setCanvas(paddle_canvas);
love.graphics.setBlendMode("alpha");
love.graphics.setColor(1, 1, 1, 1);
love.graphics.rectangle("fill", 0, 0, length, 2);
love.graphics.setCanvas();
```

All we're doing here is setting up a Canvas and filling it with a white rectangle. If you want to break this down more, go ahead and read the [love.graphics documentation](https://love2d.org/wiki/love.graphics).

Now we need to attach the canvas to the CanvasComponent.

```ts
const canvas_component = paddle_entity.add_component(CanvasComponent);
canvas_component.canvas = paddle_canvas;
canvas_component.x_scale = 1;
canvas_component.y_scale = 1;
```

Finally, let's set up its position.

```ts
const position_component = paddle_entity.add_component(PositionComponent);
position_component.x = 40;
position_component.y = 40;
```

Our final **game/game.ts** should look like this:

```ts
import { World, WorldBuilder } from "encompass-ecs";
import { CanvasComponent } from "./components/canvas";
import { PositionComponent } from "./components/position";
import { CanvasRenderer } from "./renderers/canvas";

export class Game {
    private world: World;
    private canvas: Canvas;

    public load() {
        this.canvas = love.graphics.newCanvas();

        const world_builder = new WorldBuilder();

        // ADD YOUR ENGINES HERE...

        // ADD YOUR RENDERERS HERE...
        world_builder.add_renderer(CanvasRenderer);

        // ADD YOUR STARTING ENTITIES HERE...
        const paddle_entity = world_builder.create_entity();

        const width = 4;
        const height = 8;

        const paddle_canvas = love.graphics.newCanvas(width, height);
        love.graphics.setCanvas(paddle_canvas);
        love.graphics.setBlendMode("alpha");
        love.graphics.setColor(1, 1, 1, 1);
        love.graphics.rectangle("fill", 0, 0, width, height);
        love.graphics.setCanvas();

        const canvas_component = paddle_entity.add_component(CanvasComponent);
        canvas_component.canvas = paddle_canvas;
        canvas_component.x_scale = 1;
        canvas_component.y_scale = 1;

        const position_component = paddle_entity.add_component(PositionComponent);
        position_component.x = 40;
        position_component.y = 40;

        this.world = world_builder.build();
    }

    public update(dt: number) {
        this.world.update(dt);
    }

    public draw() {
        love.graphics.clear();
        love.graphics.setCanvas(this.canvas);
        love.graphics.clear();
        this.world.draw();
        love.graphics.setCanvas();
        love.graphics.setBlendMode("alpha", "premultiplied");
        love.graphics.setColor(1, 1, 1, 1);
        love.graphics.draw(this.canvas);
    }
}
```

Let's run the game!!
