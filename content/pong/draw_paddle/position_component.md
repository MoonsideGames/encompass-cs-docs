---
title: "Position Component"
date: 2019-05-23T11:34:58-07:00
weight: 10
---

This one is pretty simple. We can't draw something if we don't know *where* on screen to draw it.

Well, why didn't we put that in the CanvasComponent? The reason is that position is a concept that is relevant in more situations than just drawing. For example: collision, yeah? So it really needs to be its own component.

Create a file: **game/components/position.ts**

```ts
import { Component } from "encompass-ecs";

export class PositionComponent extends Component {
    public x: number;
    public y: number;
}
```

That's it! Notice that we haven't created a file that is more than 10 lines long yet. I hope you're starting to notice the power of modularity here.
