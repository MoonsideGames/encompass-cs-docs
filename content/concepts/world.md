---
title: "World"
date: 2019-05-22T12:51:08-07:00
weight: 100
---

World is the pie crust that contains all the delicious Encompass ingredients together.

The World's *update* function drives the simulation and should be controlled from your engine's update loop.

The World's *draw* function tells the Renderers to draw the scene.

In LÖVE, the starter project game loop looks like this:

```ts
export class Game {
    private world: World;
    private canvas: Canvas;

    ...

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

But you can call these methods wherever you see fit.

{{% notice tip %}}
Certain Encompass projects actually have multiple separate Worlds to manage certain behaviors. This is perfectly valid and can be a great way to structure your project, but be warned that it is difficult to share information between Worlds by design.
{{% /notice %}}

That's it! Now that we have these high-level concepts down, let's build an actual, for-real game.
