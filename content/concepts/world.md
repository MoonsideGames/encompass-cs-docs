---
title: "World"
date: 2019-05-22T12:51:08-07:00
weight: 100
---

World is the pie crust that contains all the delicious Encompass ingredients together.

The World's *Update* function drives the simulation and should be controlled from your engine's update loop.

The World's *Draw* function tells the Renderers to draw the scene.

In MonoGame, the game loop looks something like this:

```cs
using Encompass;
using Microsoft.Xna.Framework;

public class MyGame : Game
{
    private World world;
    SpriteBatch spriteBatch;

    RenderTarget2D gameRenderTarget;
    RenderTarget2D levelBrowserRenderTarget;
    RenderTarget2D uiRenderTarget;

    ...

    /// <summary>
    /// Allows the game to run logic such as updating the world,
    /// checking for collisions, gathering input, and playing audio.
    /// </summary>
    /// <param name="gameTime">Provides a snapshot of timing values.</param>
    protected override void Update(GameTime gameTime)
    {
        if (GamePad.GetState(Microsoft.Xna.Framework.PlayerIndex.One).Buttons.Back == ButtonState.Pressed || Keyboard.GetState().IsKeyDown(Keys.Escape))
            Exit();

        world.Update(gameTime.ElapsedGameTime.TotalSeconds);

        base.Update(gameTime);
    }

    /// <summary>
    /// This is called when the game should draw itself.
    /// </summary>
    /// <param name="gameTime">Provides a snapshot of timing values.</param>
    protected override void Draw(GameTime gameTime)
    {
        world.Draw();

        GraphicsDevice.SetRenderTarget(null);

        spriteBatch.Begin(SpriteSortMode.Deferred, null, SamplerState.PointClamp);
        spriteBatch.Draw(gameRenderTarget, windowDimensions, Color.White);
        spriteBatch.Draw(levelBrowserRenderTarget, windowDimensions, Color.White);
        spriteBatch.Draw(uiRenderTarget, windowDimensions, Color.White);
        spriteBatch.End();

        base.Draw(gameTime);
    }
}
```

But you can call these methods wherever you see fit.

{{% notice tip %}}
Certain Encompass projects actually have multiple separate Worlds to manage certain behaviors. This is perfectly valid and can be a great way to structure your project, but be warned that it is difficult to share information between Worlds by design.
{{% /notice %}}

**What's that whole dt business about?**

*dt* stands for delta-time. Correct usage of delta-time is crucial to make sure that your game does not become *frame-dependent*, which is very bad. We'll talk more about frame-dependence later in the tutorial, but to briefly summarize, if your game is frame-dependent you will run into very frustrating behavior when running your game on different computer systems.

Even if you lock your game to a fixed timestep, writing your game with delta-time in mind can be the difference between changing the timestep being a one-line tweak or a weeks long hair-pulling nightmare.

That's it! Now that we have these high-level concepts down, let's build an actual, for-real game.
