---
title: "Putting It All Together"
date: 2019-05-28T21:22:44-07:00
weight: 1000
---

Finally, we need to set up our initial game state with our spawn messages, and make sure we added and initialized all of our required Engines.

Our load method in **game/game.ts** should look something like this:

```ts
    public load() {
        this.canvas = love.graphics.newCanvas();

        const collision_world = CollisionWorld.newWorld(32);

        const world_builder = new WorldBuilder();

        // ADD YOUR ENGINES HERE...
        world_builder.add_engine(BallSpawner).initialize(collision_world);
        world_builder.add_engine(GameBoundarySpawner).initialize(collision_world);
        world_builder.add_engine(PaddleSpawner).initialize(collision_world);

        world_builder.add_engine(InputEngine);
        world_builder.add_engine(PaddleMovementEngine);
        world_builder.add_engine(MotionEngine);
        world_builder.add_engine(VelocityEngine);

        world_builder.add_engine(CollisionCheckEngine).initialize(collision_world);
        world_builder.add_engine(CollisionDispatchEngine);
        world_builder.add_engine(BallWallCollisionEngine);
        world_builder.add_engine(BallPaddleCollisionEngine);

        world_builder.add_engine(UpdatePositionEngine);
        world_builder.add_engine(UpdateVelocityEngine);

        // ADD YOUR RENDERERS HERE...
        world_builder.add_renderer(CanvasRenderer);

        // ADD YOUR STARTING ENTITIES HERE...

        const play_area_width = 1280;
        const play_area_height = 720;

        const boundary_width = 30;

        const paddle_width = 20;
        const paddle_height = 120;
        const paddle_spacing = 40;
        const paddle_speed = 400;

        const ball_size = 16;

        const paddle_spawn_message = world_builder.emit_message(PaddleSpawnMessage);
        paddle_spawn_message.x = paddle_spacing;
        paddle_spawn_message.y = play_area_height * 0.5;
        paddle_spawn_message.width = paddle_width;
        paddle_spawn_message.height = paddle_height;
        paddle_spawn_message.move_speed = paddle_speed;

        const ball_spawn_message = world_builder.emit_message(BallSpawnMessage);
        ball_spawn_message.x = play_area_width * 0.5;
        ball_spawn_message.y = play_area_height * 0.5;
        ball_spawn_message.size = ball_size;
        ball_spawn_message.x_velocity = 200;
        ball_spawn_message.y_velocity = -400;

        const top_wall_spawn_message = world_builder.emit_message(GameBoundarySpawnMessage);
        top_wall_spawn_message.x = play_area_width * 0.5;
        top_wall_spawn_message.y = -boundary_width * 0.5;
        top_wall_spawn_message.width = play_area_width;
        top_wall_spawn_message.height = boundary_width;

        const right_wall_spawn_message = world_builder.emit_message(GameBoundarySpawnMessage);
        right_wall_spawn_message.x = play_area_width + boundary_width * 0.5;
        right_wall_spawn_message.y = play_area_height * 0.5;
        right_wall_spawn_message.width = boundary_width;
        right_wall_spawn_message.height = play_area_height;

        const bottom_wall_spawn_message = world_builder.emit_message(GameBoundarySpawnMessage);
        bottom_wall_spawn_message.x = play_area_width * 0.5;
        bottom_wall_spawn_message.y = boundary_width * 0.5 + play_area_height;
        bottom_wall_spawn_message.width = play_area_width;
        bottom_wall_spawn_message.height = boundary_width;

        const left_wall_spawn_message = world_builder.emit_message(GameBoundarySpawnMessage);
        left_wall_spawn_message.x = -boundary_width * 0.5;
        left_wall_spawn_message.y = play_area_height * 0.5;
        left_wall_spawn_message.width = boundary_width;
        left_wall_spawn_message.height = play_area_height;

        this.world = world_builder.build();
    }
```

Let's try it!

```sh
npm run love
```

<video width="75%" height="360" autoplay="autoplay" muted="muted" loop="loop" style="display: block; margin: 0 auto;">
    <source src="/images/bouncing.webm" type="video/webm">
</video>

All our hard work paid off. Look at that! *chef kiss*
