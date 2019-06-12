---
title: "Canvas Renderer"
date: 2019-05-23T11:29:24-07:00
weight: 10
---

Now that we have a CanvasComponent, we need to tell Encompass how to draw things that have it.

Create a file: **game/renderers/canvas.ts**

This is gonna be a bit more complex than our Components, so let's take this slowly.

```ts
import { Entity, EntityRenderer } from "encompass-ecs";
import { CanvasComponent } from "game/components/canvas";
import { PositionComponent } from "game/components/position";

@Renders(CanvasComponent, PositionComponent)
export class CanvasRenderer extends EntityRenderer {
    public render(entity: Entity) {}
}
```

An *EntityRenderer* is defined by the Components it tracks and a *render* method.

**@Renders** is a function called a *class decorator*. Its first argument should be the DrawComponent, and the subsequent arguments are any number of components that are also required for the EntityRenderer to *track* an Entity.

{{% notice tip %}}
You can read more about decorators on the [official TypeScript documentation](https://www.typescriptlang.org/docs/handbook/decorators.html).
{{% /notice %}}

Each time *World.draw* is called, the EntityRenderer will run its *render* method on each Entity that it is tracking.

So, in our case, we want our CanvasRenderer to render any Entity that has a PositionComponent and a CanvasComponent. Simple as that.

Let's fill out our *render* method.

```ts
    public render(entity: Entity) {
        const position_component = entity.get_component(PositionComponent);
        const canvas_component = entity.get_component(CanvasComponent);

        const canvas = canvas_component.canvas;

        love.graphics.draw(
            canvas,
            position_component.x,
            position_component.y,
            0,
            canvas_component.x_scale,
            canvas_component.y_scale,
            canvas.getWidth() * 0.5,
            canvas.getHeight() * 0.5,
        );
    }
```

*Entity.get_component* is a method that gets a Component instance from an Entity when given a Component type. So when we say:

```ts
const position_component = entity.get_component(PositionComponent);
```

we are asking the Entity to give us access to its position information.

Once we have our specific position and canvas information, we can use that information to tell LOVE to draw something!

```ts
love.graphics.draw(
    canvas,
    position_component.x,
    position_component.y,
    0,
    canvas_component.x_scale,
    canvas_component.y_scale,
    canvas.getWidth() * 0.5,
    canvas.getHeight() * 0.5,
);
```

This is simply a call to the *love.graphics.draw* function that LOVE provides. You can read more about it [here](https://love2d.org/wiki/love.graphics.draw). We are just telling LOVE to draw our canvas at our PositionComponent's position, with 0 rotation, our scaling factor, and an offset of the canvas's width and height divided by 2. The offset just tells LOVE to draw the canvas starting at the center of the canvas, instead of at the top left corner.

That's it! Now we need to set up our World with its starting configuration so our Encompass elements can work in concert.

{{% notice notice %}}
Clever readers may have noticed something here. Aren't Entities allowed to have any number of Components of a given type? So why is *get_component* singular?

We actually have two different component getter methods: *Entity.get_component*, and *Entity.get_components*, which will return a list of all the components of the given type on the Entity.

In this case, I am assuming that an Entity will only ever have one PositionComponent, so I am using the *get_component* method for convenience.

You are allowed to make any assumptions about the structure of your Entities as you want - just make sure your assumptions stay consistent, or you will have unpleasant surprises!
{{% /notice %}}
