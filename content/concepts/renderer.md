---
title: "Renderer"
date: 2019-05-22T14:16:06-07:00
weight: 30
---

A Renderer is responsible for reading the game state and telling the game engine what to draw to the screen.

{{% notice notice %}}
Remember: Encompass isn't a game engine and it doesn't have a rendering system. So Renderers aren't actually doing the rendering, they're just telling the game engine what to render.
{{% /notice %}}

There are two kinds of renderers: GeneralRenderers and EntityRenderers.

A GeneralRenderer is a Renderer which reads the game state in order to draw elements to the screen. It also requires a layer, which represents the order in which it will draw to the screen.

If you were using the LOVE engine, a GeneralRenderer might look like this:

```ts
import { GeneralRenderer } from "encompass-ecs";
import { ScoreComponent } from "game/components/score";

export class ScoreRenderer extends GeneralRenderer {
    public layer = 4;

    public render() {
        const score_component = this.read_component(ScoreComponent);

        love.graphics.print(score_component.score, 20, 20);
    }
}
```

An EntityRenderer provides a structure for the common pattern of drawing an Entity which has a particular collection of Components and a specific type of DrawComponent. They also have the ability to draw DrawComponents at their specific layer.

If you were using the LOVE engine, a GeneralRenderer might look like this:

```ts
import { EntityRenderer } from "encompass-ecs";
import { PointComponent } from "game/components/point";
import { PositionComponent } from "game/components/position";

@Renders(PointComponent, PositionComponent)
export class PointRenderer extends EntityRenderer {
    public render(entity: Entity) {
        const point_component = entity.get_component(PointComponent);
        const position_component = entity.get_component(PositionComponent);

        const color = point_component.color;
        love.graphics.setColor(color.r, color.g, color.b, color.a);
        love.graphics.point(position_component.x, position_component.y);
    }
}
```

For 2D games, you will need to use layers to be specific about the order in which entities draw. For a 3D game you will probably end up delegating rendering to some kind of scene/camera system.
