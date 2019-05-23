---
title: "Canvas Component"
date: 2019-05-23T11:26:31-07:00
weight: 5
---

LOVE provides a neat little drawing feature called Canvases. You can tell LOVE to draw to a Canvas instead of the screen, and then save the Canvas so you don't have to repeat lots of draw procedures. It's very nifty.

Let's set up a CanvasComponent.

Create a file: **game/components/canvas.ts**

```ts
import { DrawComponent } from "encompass-ecs";

export class CanvasComponent extends DrawComponent {
    public canvas: Canvas;
    public x_scale: number;
    public y_scale: number;
}
```

Let's break this down a bit. What's a DrawComponent? A DrawComponent is a subtype of Component that includes a *layer* property, which is used for rendering.

*import* means that we are taking the definition of DrawComponent from another file, in this case the Encompass library. *export* means that we want this class to be available to other files in our project. If we don't export this class, it won't be very useful to us, so let's make sure to do that.

We provide some extra information, *x_scale* and *y_scale* so we can shrink or stretch the Canvas if we want to.

{{% notice notice %}}
You might be wondering - how does TypeScript know about things like Canvas, which are defined in LOVE? LOVE uses Lua, not TypeScript.

The answer is a thing called *definition files*. Definition files let TypeScript know about things that exist in the target environment. You don't really need to understand how it works just now, just know that the Encompass/LOVE starter pack depends on the lovely [love-typescript-definitions](https://github.com/hazzard993/love-typescript-definitions) project.
{{% /notice %}}

When we actually use the CanvasComponent, we will attach a Canvas that has stuff drawn on it. We'll get to that in a minute.

That's it for our CanvasComponent. We need one more bit of information before we can write our Renderer.
