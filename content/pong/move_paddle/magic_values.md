---
title: "Magic Values"
date: 2019-05-23T15:51:58-07:00
weight: 25
---

Our code right now is violating one more good architecture principle.

```ts
if (love.keyboard.isDown("up")) {
    const message = this.emit_component_message(MotionMessage, player_one_position_component);
    message.x = 0;
    message.y = -10;
}
```

*Magic values* refer to numbers that have been placed directly in the code. The values being set to *message.x* and *message.y* above are magic values. Why are magic values bad?

Magic values introduce the possibility of duplication. Let's say I start adding my code to be able to move paddles down. Now I have to change the numbers in two different places. If I ever change one without changing the other, I have introduced a bug.

There's another reason to avoid magic values too. Suppose I haven't looked at the InputEngine for a while, but I suddenly decide that the paddles are moving too slow. Intuitively, I would want to look for a Component that contains those values, but instead, they would be hidden in the InputEngine. This isn't what you would really expect - why would Input have anything to do with the speed of the paddles?

Organizing your information consistently is crucial to being able to easily find things that you need to change. You'll thank yourself later.

Let's make a new Component.

Create a file: **game/components/paddle_move_speed.ts**

```ts
import { Component } from "encompass-ecs";

export class PaddleMoveSpeedComponent extends Component {
    public y: number;
}
```

And let's add it to our paddle Entity.

In **game/game.ts**

```ts
const move_speed_component = paddle_entity.add_component(PaddleMoveSpeedComponent);
move_speed_component.y = 10;
```

Now let's tell our InputEngine to use it, and why don't we go ahead and make the "down" key move the paddle downward too.

```ts
...

if (player_one_entity) {
    const player_one_position_component = player_one_entity.get_component(PositionComponent);
    const player_one_move_speed_component = player_one_entity.get_component(PaddleMoveSpeedComponent);

    if (love.keyboard.isDown("up")) {
        const message = this.emit_component_message(MotionMessage, player_one_position_component);
        message.x = 0;
        message.y = -player_one_move_speed_component.y;
    } else if (love.keyboard.isDown("down")) {
        const message = this.emit_component_message(MotionMessage, player_one_position_component);
        message.x = 0;
        message.y = player_one_move_speed_component.y;
    }
}

...
```

I'm starting to get get a bad feeling about this. Are you? Let me explain.
