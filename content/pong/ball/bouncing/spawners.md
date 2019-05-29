---
title: "Spawners"
date: 2019-05-29T11:05:16-07:00
weight: 900
---

Our Entities are getting a bit more complex now with the addition of BoundingBoxComponents and CollisionTypeComponents.

I think we should create Spawners for each of our game entities.

This will be pretty straightforward. Just decide which parameters we need to create our entities, and add the proper components with those parameters.

### Ball

In **game/messages/ball_spawn.ts**:

```ts
import { Message } from "encompass-ecs";

export class BallSpawnMessage extends Message {
    public x: number;
    public y: number;
    public size: number;
    public x_velocity: number;
    public y_velocity: number;
}
```

In **game/engines/spawners/ball.ts**:

```ts
import { Reads, Spawner } from "encompass-ecs";
import { BoundingBoxComponent } from "game/components/bounding_box";
import { CanvasComponent } from "game/components/canvas";
import { CollisionType, CollisionTypesComponent } from "game/components/collision_types";
import { PositionComponent } from "game/components/position";
import { VelocityComponent } from "game/components/velocity";
import { BallSpawnMessage } from "game/messages/ball_spawn";
import { World } from "lua-lib/bump";

@Reads(BallSpawnMessage)
export class BallSpawner extends Spawner {
    public spawn_message_type = BallSpawnMessage;

    public collision_world: World;

    public initialize(collision_world: World) {
        this.collision_world = collision_world;
    }

    public spawn(message: BallSpawnMessage) {
        const ball_entity = this.create_entity();

        const ball_position_component = ball_entity.add_component(PositionComponent);
        ball_position_component.x = message.x;
        ball_position_component.y = message.y;

        const ball_canvas = love.graphics.newCanvas(message.size, message.size);
        love.graphics.setCanvas(ball_canvas);
        love.graphics.setBlendMode("alpha");
        love.graphics.setColor(1, 1, 1, 1);
        love.graphics.rectangle("fill", 0, 0, message.size, message.size);
        love.graphics.setCanvas();

        const ball_canvas_component = ball_entity.add_component(CanvasComponent);
        ball_canvas_component.canvas = ball_canvas;
        ball_canvas_component.x_scale = 1;
        ball_canvas_component.y_scale = 1;

        const velocity_component = ball_entity.add_component(VelocityComponent);
        velocity_component.x = message.x_velocity;
        velocity_component.y = message.y_velocity;

        const boundaries_component = ball_entity.add_component(BoundingBoxComponent);
        boundaries_component.width = message.size;
        boundaries_component.height = message.size;

        const collision_types_component = ball_entity.add_component(CollisionTypesComponent);
        collision_types_component.collision_types = [ CollisionType.ball ];

        this.collision_world.add(ball_entity, message.x, message.y, message.size, message.size);
    }
}
```

### Game Boundary

In **game/messages/game_boundary_spawn.ts**:

```ts
import { Message } from "encompass-ecs";

export class GameBoundarySpawnMessage extends Message {
    public x: number;
    public y: number;
    public width: number;
    public height: number;
}
```

In **game/spawners/game_boundary.ts**:

```ts
import { Reads, Spawner } from "encompass-ecs";
import { BoundingBoxComponent } from "game/components/bounding_box";
import { CollisionType, CollisionTypesComponent } from "game/components/collision_types";
import { PositionComponent } from "game/components/position";
import { GameBoundarySpawnMessage } from "game/messages/game_boundary_spawn";
import { World } from "lua-lib/bump";

@Reads(GameBoundarySpawnMessage)
export class GameBoundarySpawner extends Spawner {
    public spawn_message_type = GameBoundarySpawnMessage;

    private collision_world: World;

    public initialize(collision_world: World) {
        this.collision_world = collision_world;
    }

    public spawn(message: GameBoundarySpawnMessage) {
        const entity = this.create_entity();

        const boundaries = entity.add_component(BoundingBoxComponent);
        boundaries.width = message.width;
        boundaries.height = message.height;

        const position = entity.add_component(PositionComponent);
        position.x = message.x;
        position.y = message.y;

        const collision_types_component = entity.add_component(CollisionTypesComponent);
        collision_types_component.collision_types = [ CollisionType.wall ];

        this.collision_world.add(
            entity,
            message.x - message.width * 0.5,
            message.y - message.height * 0.5,
            message.width,
            message.height
        );
    }
}
```

### Paddle

In **game/messages/paddle_spawn.ts**:

```ts
import { Message } from "encompass-ecs";

export class PaddleSpawnMessage extends Message {
    public x: number;
    public y: number;
    public width: number;
    public height: number;
    public move_speed: number;
}
```

In **game/spawners/paddle.ts**:

```ts
import { Reads, Spawner } from "encompass-ecs";
import { BoundingBoxComponent } from "game/components/bounding_box";
import { CanvasComponent } from "game/components/canvas";
import { CollisionType, CollisionTypesComponent } from "game/components/collision_types";
import { PaddleMoveSpeedComponent } from "game/components/paddle_move_speed";
import { PlayerOneComponent } from "game/components/player_one";
import { PositionComponent } from "game/components/position";
import { PaddleSpawnMessage } from "game/messages/paddle_spawn";
import { World } from "lua-lib/bump";

@Reads(PaddleSpawnMessage)
export class PaddleSpawner extends Spawner {
    public spawn_message_type = PaddleSpawnMessage;

    private collision_world: World;

    public initialize(collision_world: World) {
        this.collision_world = collision_world;
    }

    protected spawn(message: PaddleSpawnMessage) {
        const paddle_entity = this.create_entity();

        paddle_entity.add_component(PlayerOneComponent);

        const width = message.width;
        const height = message.height;

        const paddle_canvas = love.graphics.newCanvas(width, height);
        love.graphics.setCanvas(paddle_canvas);
        love.graphics.setBlendMode("alpha");
        love.graphics.setColor(1, 1, 1, 1);
        love.graphics.rectangle("fill", 0, 0, width, height);
        love.graphics.setCanvas();

        const canvas_component = paddle_entity.add_component(CanvasComponent);
        canvas_component.canvas = paddle_canvas;
        canvas_component.x_scale = 1;
        canvas_component.y_scale = 1;

        const position_component = paddle_entity.add_component(PositionComponent);
        position_component.x = message.x;
        position_component.y = message.y;

        const move_speed_component = paddle_entity.add_component(PaddleMoveSpeedComponent);
        move_speed_component.y = message.move_speed;

        const paddle_boundaries = paddle_entity.add_component(BoundingBoxComponent);
        paddle_boundaries.width = width;
        paddle_boundaries.height = height;

        const collision_types_component = paddle_entity.add_component(CollisionTypesComponent);
        collision_types_component.collision_types = [ CollisionType.paddle ];

        this.collision_world.add(
            paddle_entity,
            message.x - width * 0.5,
            message.y - height * 0.5,
            width,
            height
        );
    }
}
```
