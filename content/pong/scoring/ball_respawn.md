---
title: "Ball Respawn"
date: 2019-06-04T10:44:53-07:00
weight: 20
---

In Pong, when the ball collides with the goal, we want it to respawn after a set amount of time, fired from a random point in the center of the play area in a variable direction.

We can easily implement a timer by using a Component.

Let's create a new Entity: a timed ball spawner.

You should be getting fairly familiar with this process by now. We'll need a Component, a Message, and a Spawner.

In **game/components/ball_spawn_timer.ts**:

```ts
import { Component } from "encompass-ecs";

export class BallSpawnTimerComponent extends Component {
    public time_remaining: number;
}
```

In **game/messages/ball_spawn_timer_spawn.ts**:

```ts
import { Message } from "encompass-ecs";

export class BallSpawnTimerSpawnMessage extends Message {
    public time: number;
}
```

In **game/engines/spawners/ball_spawn_timer.ts**:

```ts
import { Reads, Spawner } from "encompass-ecs";
import { BallSpawnTimerComponent } from "game/components/ball_spawn_timer";
import { BallSpawnTimerSpawnMessage } from "game/messages/ball_spawn_timer_spawn";

@Reads(BallSpawnTimerSpawnMessage)
export class BallSpawnTimerSpawner extends Spawner {
    public spawn_message_type = BallSpawnTimerSpawnMessage;

    public spawn(message: BallSpawnTimerSpawnMessage) {
        const entity = this.create_entity();

        const component = entity.add_component(BallSpawnTimerComponent);
        component.time_remaining = message.time;
    }
}
```

Finally we need an Engine to control the timer behavior and the firing of the ball spawn message.

When we serve the ball in Pong, it fires from a random position along the center line at a random angle (with some constraints). Sound's like we're gonna need some vector math.

If you don't know anything about vectors, a 2D vector is simply a mathematical structure composed of an _x_ and _y_ component. We generally use them to represent both position and velocity. There are certain clever mathematical operations we can do on vectors that make them very useful for games.

It turns out there is a very useful Lua library for 2D vector math in a repository called HUMP. Download it [here](https://github.com/vrld/hump/blob/master/vector-light.lua).

Create a new directory, **lua-lib/hump**, and add vectorlight.lua to the directory. Let's write a declaration file to go along with it.

In **lua-lib/hump/vectorlight.d.ts**:

```ts
/** @noSelfInFile */

export function str(x: number, y: number): string;

/** @tupleReturn */
export function mul(s: number, x: number, y: number): [number, number];

/** @tupleReturn */
export function div(s: number, x: number, y: number): [number, number];

/** @tupleReturn */
export function add(x1: number, y1: number, x2: number, y2: number): [number, number];

/** @tupleReturn */
export function sub(x1: number, y1: number, x2: number, y2: number): [number, number];

/** @tupleReturn */
export function permul(x1: number, y1: number, x2: number, y2: number): [number, number];

export function dot(x1: number, y1: number, x2: number, y2: number): number;

export function det(x1: number, y1: number, x2: number, y2: number): number;

export function eq(x1: number, y1: number, x2: number, y2: number): boolean;

export function lt(x1: number, y1: number, x2: number, y2: number): boolean;

export function le(x1: number, y1: number, x2: number, y2: number): boolean;

export function len2(x: number, y: number): number;

export function len(x: number, y: number): number;

/** @tupleReturn */
export function fromPolar(angle: number, radius: number): [number, number];

/** @tupleReturn */
export function randomDirection(len_min: number, len_max: number): [number, number];

/** @tupleReturn */
export function toPolar(x: number, y: number): [number, number];

export function dist2(x1: number, y1: number, x2: number, y2: number): number;

export function dist(x1: number, y1: number, x2: number, y2: number): number;

/** @tupleReturn */
export function normalize(x: number, y: number): [number, number];

/** @tupleReturn */
export function rotate(phi: number, x: number, y: number): [number, number];

/** @tupleReturn */
export function perpendicular(x: number, y: number): [number, number];

/** @tupleReturn */
export function project(x: number, y: number, u: number, v: number): [number, number];

/** @tupleReturn */
export function mirror(x: number, y: number, u: number, v: number): [number, number];

/** @tupleReturn */
export function trim(maxLen: number, x: number, y: number): [number, number];

export function angleTo(x: number, y: number, u: number, v: number): number;
```

Now we can use all these useful vector math functions in our game.


Let's break down some of the math here.

We want the ball to fire in a random direction, but we don't want its trajectory to be too vertical, or it will take forever to get to one of the paddles, which is boring. We also want it to travel at the same speed regardless of its direction.

Let's start with a vector with an _x_ component of 0 and a _y_ component of our desired speed. You can think of this as an arrow pointing straight downward. Now imagine that arrow as the hand of a clock. How can we describe the angle of the clock hand? As the hand rotates around it differs from its original orientation in an amount of units called _radians_. When that angle changes by 2 times pi, it ends up in the same position. So a rotation a quarter of the way around the clock would be pi divided by 2.

![pi over 2](/images/pi-over-2.png)

You can see that if we rotate our downward vector by pi/2 radians, it will face towards the left paddle. Now, what we want is for the ball to be served at angles like this:

![serve area](/images/serve-area.png)

The non-shaded area represents the angles that we want the ball to be served at. What angle is that exactly?

Well, a lot of what we do in game math is guesstimation. "Close enough" can be a powerful phrase! We can always easily tweak the exact values later if we architect our game properly.

![serve area angles](/images/serve-area-angles.png)

If we draw it out, we know that a quarter-circle rotation is pi/2 radians. The start of our serve range seems to be roughly half that. So that would be pi/4 radians. Sounds reasonable as a starting angle to me. What about the ending angle?

![three quarters circle](/images/three-quarters.png)

When we draw it out, we see that it is 3 eighth-circle rotations. So we get 3 * pi/4.

So now we need to pick a random rotation within this range. How should we do that? TypeScript and Lua don't have anything built-in for this. I usually write a helper for this, since it's so common to want a random real number in a certain range.

Let's create **game/helpers/math.ts**:

```ts
export class MathHelper {
    public static randomFloat(low: number, high: number): number {
        return love.math.random() * high + low;
    }
}
```
_love.math.random()_ returns a random real number between 0 and 1. So our _randomFloat_ function will return a random real number between _low_ and _high_.

One last note about rotations. What if the rotation is _negative_? Well, our positive rotations have been going clockwise - so negative rotations go counter-clockwise!

Now we can construct a formula for our random serve direction.

```ts
const direction = MathHelper.randomFloat(math.pi / 4, math.pi * 3 / 4) *
                    (love.math.random() > 0.5 ? 1 : -1);
```

What's that last bit on the second line? Remember, _love.math.random()_ returns a random number between 0 and 1. It has a 50% chance of being greater than 0.5. So that last expression means, there's a 50% chance of that value being equal to 1, and a 50% chance of it being equal to -1. If we multiply the rotation by negative 1, we are reversing its direction, so we have an equal chance of the ball being served to the left or the right. Spiffy!

Also, let's remember to destroy our timer entity at the end so it doesn't keep firing events. That would be bad!

Let's put it all together.

In **game/engines/ball_spawn_timer.ts**:

```ts
import { Emits, Engine, Mutates } from "encompass-ecs";
import { BallSpawnTimerComponent } from "game/components/ball_spawn_timer";
import { MathHelper } from "game/helpers/math";
import { BallSpawnMessage } from "game/messages/ball_spawn";
import * as vectorlight from "lua-lib/hump/vectorlight";

@Mutates(BallSpawnTimerComponent)
@Emits(BallSpawnMessage)
export class BallSpawnTimerEngine extends Engine {
    private ball_size: number;
    private ball_speed: number;
    private min_serve_angle: number;
    private max_serve_angle: number;
    private middle: number;
    private height: number;

    public initialize(
        ball_size: number,
        ball_speed: number,
        min_serve_angle: number,
        max_serve_angle: number,
        middle: number,
        height: number,
    ) {
        this.ball_size = ball_size;
        this.ball_speed = ball_speed;
        this.min_serve_angle = min_serve_angle;
        this.max_serve_angle = max_serve_angle;
        this.middle = middle;
        this.height = height;
    }

    public update(dt: number) {
        for (const component of this.read_components_mutable(BallSpawnTimerComponent).values()) {
            component.time_remaining -= dt;

            if (component.time_remaining <= 0) {
                const ball_spawn_message = this.emit_message(BallSpawnMessage);
                ball_spawn_message.x = this.middle;
                ball_spawn_message.y = love.math.random() * this.height;

                const direction = MathHelper.randomFloat(
                    this.min_serve_angle,
                    this.max_serve_angle,
                ) * (love.math.random() > 0.5 ? 1 : -1);

                [
                    ball_spawn_message.x_velocity,
                    ball_spawn_message.y_velocity,
                ] = vectorlight.rotate(direction, 0, this.ball_speed);

                ball_spawn_message.size = this.ball_size;

                this.get_entity(component.entity_id)!.destroy();
            }
        }
    }
}
```

Every frame we subtract the remaining time by the delta-time value. Once it less than or equal to zero, we fire a BallSpawnMessage and destroy the timer entity.

Don't forget to register our new Engines with the WorldBuilder.

```ts
world_builder.add_engine(BallSpawnTimerSpawner);
world_builder.add_engine(BallSpawnTimerEngine).initialize(
    ball_size,
    ball_speed,
    math.pi / 4,
    3 * math.pi / 4,
    play_area_width * 0.5,
    play_area_height,
);
```

While we're in **game.ts**, let's also change our game start ball spawn to use our fancy new timer based ball spawn system.

```ts
const ball_timer_spawn_message = world_builder.emit_message(BallSpawnTimerSpawnMessage);
ball_timer_spawn_message.time = 1;
```

The moment of truth...

<video width="640" height="360" autoplay="autoplay" muted="muted" loop="loop" style="display: block; margin: 0 auto; width: 640;">
    <source src="/images/ball_respawn.webm" type="video/webm">
</video>

Not bad. The angle is probably a bit generous as it is but we can leave it for now.
