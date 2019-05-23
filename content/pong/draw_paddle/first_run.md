---
title: "First Run"
date: 2019-05-23T12:21:05-07:00
weight: 20
---

All we have to do now is run our build and run script in the terminal.

```sh
> npm run love
```

Exciting!! Let's see what happens...

![pong first run](/images/pong_first_run.png)

Oh dear. That paddle is quite small. Bit of a buzzkill really.

```ts
const width = 20;
const height = 120;
```

```ts
position_component.x = 40;
position_component.y = 360;
```

![pong second run](/images/pong_second_run.png)

Thaaaaaaat's more like it.

Notice how we can just change simple Component values, and the result of the simulation changes. In a larger project we would probably want to define these Component values in a separate file that lives on its own. This is called *data-driven design* and it is a powerful feature of ECS-like architectures. When we do data-driven design, we can modify the game without even looking at source code - just change some values in a file and the game changes! If we wanted to get really clever, we could probably have an in-game editor that changes these values even while the game is running!

But for such a simple example, leaving this in the *load* function is probably fine. Let's move on and get this paddle moving.
