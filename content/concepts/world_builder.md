---
title: "World Builder"
date: 2019-05-22T13:40:25-07:00
weight: 35
---

WorldBuilder is used to construct a World from Engines, Renderers, and an initial state of Entities, Components, and Messages.

The WorldBuilder enforces certain rules about Engine structure. It is forbidden to have messages create cycles between Engines, and no Component may be mutated by more than one Engine.

The WorldBuilder uses Engines and their Message read/emit information to determine a valid ordering of the Engines, which is given to the World.

Here is an example usage:

```ts
class Game {
    private World world;

    ...

    public load() {
        import { WorldBuilder } from "encompass-ecs";
        import { CanvasComponent } from "./components/canvas";
        import { PositionComponent } from "./components/position";
        import { VelocityComponent } from "./components/velocity";
        import { MotionEngine } from "./engines/motion";
        import { CanvasRenderer } from "./renderers/canvas";

        const world_builder = new WorldBuilder();

        world_builder.add_engine(MotionEngine);
        world_builder.add_renderer(CanvasRenderer);

        const entity = world_builder.create_entity();

        const position_component = entity.add_component(PositionComponent);
        position_component.x = 0;
        position_component.y = 0;

        const velocity_component = entity.add_component(VelocityComponent);
        velocity_component.x = 20;
        velocity_component.y = 0;

        const sprite_component = entity.add_component(SpriteComponent);
        canvas_component.canvas = love.graphics.newImage("assets/sprites/ball.png");

        this.world = world_builder.build();
    }

    ...
}
```

Now our game will initialize with a ball that moves horizontally across the screen!

{{% notice tip %}}
Make sure that you remember to add Engines to the WorldBuilder when you define them. Otherwise nothing will happen, which can be very embarrassing.
{{% /notice %}}
