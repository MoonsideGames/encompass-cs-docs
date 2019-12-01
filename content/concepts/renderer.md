---
title: "Renderer"
date: 2019-05-22T14:16:06-07:00
weight: 30
---

A Renderer is responsible for reading the game state and telling the game engine what to draw to the screen.

{{% notice notice %}}
Remember: Encompass isn't a game engine and it doesn't have a rendering system. So Renderers aren't actually doing the rendering,it is just a way to structure how we tell the game engine what to render.
{{% /notice %}}

There are two kinds of renderers: GeneralRenderers and OrderedRenderers.

A GeneralRenderer is a Renderer which reads the game state in order to draw elements to the screen. It also requires a layer, which represents the order in which it will draw to the screen.

If you were using MonoGame, a GeneralRenderer might look like this:

```cs
using System;
using Encompass;
using Microsoft.Xna.Framework;
using SamuraiGunn2.Components;
using SamuraiGunn2.Editor.Components;
using SamuraiGunn2.Helpers;

namespace SamuraiGunn2.Editor.Renderers
{
    public class GridRenderer : GeneralRenderer
    {
        private int gridSize;
        private PrimitiveDrawer primitiveDrawer;

        public GridRenderer(PrimitiveDrawer primitiveDrawer)
        {
            this.primitiveDrawer = primitiveDrawer;
            this.gridSize = 16;
        }

        public override void Render()
        {
            if (SomeComponent<EditorModeComponent>() && SomeComponent<MouseComponent>())
            {
                var entity = ReadEntity<MouseComponent>();
                var transformComponent = GetComponent<TransformComponent>(entity);

                Rectangle rectangle = new Rectangle
                {
                    X = (transformComponent.Position.X / gridSize) * gridSize,
                    Y = (transformComponent.Position.Y / gridSize) * gridSize,
                    Width = gridSize,
                    Height = gridSize
                };

                primitiveDrawer.DrawBorder(rectangle, 0, new System.Numerics.Vector2(1, 1), Color.White, 1);
            }
        }
    }
}
```

GeneralRenderers are great for things like UI layers, where we always want a group of particular elements to be drawn at a specific layer regardless of the specifics of the game state.

An OrderedRenderer provides a structure for the common pattern of wanting to draw an individual Component at a specific layer. OrderedRenderers must specify a component that implements IDrawableComponent.

If you were using MonoGame, an OrderedRenderer might look like this:

```cs
using Encompass;
using Microsoft.Xna.Framework.Graphics;
using SamuraiGunn2.Components;
using SamuraiGunn2.Extensions;
using System;
using System.Numerics;

namespace SamuraiGunn2.Renderers
{
    public class Texture2DRenderer : OrderedRenderer<Texture2DComponent>
    {
        private SpriteBatch spriteBatch;

        public Texture2DRenderer(SpriteBatch spriteBatch)
        {
            this.spriteBatch = spriteBatch;
        }

        public override void Render(Entity entity, Texture2DComponent textureComponent)
        {
            var transformComponent = GetComponent<TransformComponent>(entity);

            spriteBatch.Draw(textureComponent.texture, transformComponent.Position, null, textureComponent.color, transformComponent.Rotation, textureComponent.origin, transformComponent.Scale, SpriteEffects.None, 0);
        }
    }
}
```

For 2D games, you will need to use layers to be specific about the order in which elements are drawn to the screen. For a 3D game you will probably end up delegating most of the rendering to some kind of scene/camera system.
