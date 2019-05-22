---
title: "Choosing An Engine"
date: 2019-05-22T09:53:32-07:00
weight: 5
---

Encompass is not a game engine.

It does not provide any tools for drawing graphics to the screen, playing sounds, handling inputs, or anything of the sort.

Encompass is purely a tool for managing the code that handles the *simulation* aspects of your game.

That means it needs to run on top of an engine.

**Which engine should I use?**

Ultimately, this is a question that you have to answer for your project. Is there an engine you're already comfortable with? Which platforms are you targeting? Linux? PS4? Android? Are there any features you would really like to have, like a built-in physics simulator? These are questions that could help you choose an engine.

Encompass-TS can hook into any engine that supports JavaScript or Lua scripting.
Encompass-CS can hook into any engine that supports C# scripting.

So you have a lot of choices!

Here are some engines that I have used:

[LÖVE](https://love2d.org/) is a wonderful little framework for 2D games. It's cross-platform and very lightweight, but with a lot of powerful features, including a complete physics simulator. It uses Lua scripting, so you would want Encompass-TS.

[MonoGame](http://www.monogame.net/) is a cross-platform 2D/3D framework that thousands of games have used. You can use it to ship games on basically any platform that exists and it is extremely well-supported. It uses C# scripting.

[BabylonJS](https://www.babylonjs.com/) uses the power of WebGL to run 3D games in the browser. It has a powerful graphics pipeline and you can make stuff that has some wow factor. It runs on JS.

Encompass gives you the power to develop using many different engines, so feel free to experiment and find one you like! And if you switch to an engine that uses the same scripting language, it's actually very easy to switch engines, because the simulation layer is mostly self-contained.
