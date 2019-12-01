---
title: "World Builder"
date: 2019-05-22T13:40:25-07:00
weight: 35
---

WorldBuilder is used to construct a World from Engines, Renderers, and an initial state of Entities, Components, and Messages.

The WorldBuilder enforces certain rules about Engine structure. It is forbidden to have messages create cycles between Engines, and no Component may be mutated by more than one Engine without declaring a priority.

The WorldBuilder uses Engines and their Message read/send information to determine a valid ordering of the Engines, which is given to the World.

Here is an example usage with MonoGame:

```cs
using Encompass;
using Microsoft.Xna.Framework;

public class MyGame : Game
{
    private World world;

    ...

    protected override void LoadContent()
    {
        var worldBuilder = new WorldBuilder();

        worldBuilder.AddEngine(new MotionEngine());
        worldBuilder.AddEngine(new TextureRenderer());

        var entity = worldBuilder.CreateEntity();

        SetComponent(entity, new PositionComponent
        {
            x = 0,
            y = 0
        });

        SetComponent(entity, new VelocityComponent
        {
            x = 20,
            y = 0
        });

        SetComponent(entity, new TextureComponent
        {
            texture = TextureHelper.LoadTexture("assets/sprites/ball.png")
        });

        world = worldBuilder.Build();
    }

    ...
}
```

Now our game will initialize with a ball that moves horizontally across the screen!

{{% notice tip %}}
Be extra careful that you remember to add Engines to the WorldBuilder when you define them. Otherwise nothing will happen, which can be very embarrassing.
{{% /notice %}}
