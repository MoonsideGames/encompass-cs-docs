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

An EntityRenderer provides a structure for the common pattern of drawing an Entity which has a particular collection of Components and a specific type of DrawComponent. They also have the ability to draw DrawComponents at their specific layer.

For 2D games, you will need to be specific about the order in which things draw. For a 3D game you will probably end up delegating rendering to some kind of scene/camera system.
