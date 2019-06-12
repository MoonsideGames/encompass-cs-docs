---
title: "LÖVE Project Structure"
date: 2019-05-22T12:19:55-07:00
weight: 15
---

Structuring your project is a crucial component of keeping your Encompass development sane. Let's review how an Encompass project is typically structured.

If we look at the Encompass/LÖVE starter project, it looks like this:

![the directory structure of the starter project](/images/structure.png)

**main.ts** is the entry point of the game for LÖVE. You can change top-level configuration things here, like setting the window size or whether the mouse is visible. You should review the [LÖVE Documentation](https://love2d.org/wiki/Main_Page) for more clarification.

**conf.ts** defines certain information about the game. You can read more about LÖVE Config Files [here](https://love2d.org/wiki/Config_Files).

**game/game.ts** contains a class that defines three methods that are called by LOVE: load, update, and draw. They do what it says on the tin. You will set up your WorldBuilder in the load method. (We'll talk about that in a bit.)

The rest of it is pretty straightforward. Put your music and sprites and such in the **assets** folder. Define your components in the **components** folder, your engines in the **engines** folder, your messages in the **messages** folder, and your renderers in the **renderers** folder. (Again, we'll start getting into exactly how to define these in a minute.)

Finally, a quick note about **helpers**. I like to use classes with static methods for common behaviors that will be useful for many different engines, for example a `Color` class with a `hsv_to_rgb` conversion function. Be careful not to abuse helpers. If your helpers need to be instantiated, that is usually a sign that the behavior belongs in an engine.
