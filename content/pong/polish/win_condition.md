---
title: "Win Condition"
date: 2019-06-09T19:10:15-07:00
weight: 30
---

There's one critical element missing from our game. Right now, once the game starts, it keeps going until the player exits the program. We need a win condition, and to indicate when that win condition has been reached.

First let's create two separate components - one that holds information about the victory, and one that counts down to when the game should return to the title screen.

```ts
import { Component } from "encompass-ecs";

export class WinDisplayComponent extends Component {
    public player_index: number;
}
```

```ts
import { Component } from "encompass-ecs";

export class FinishGameTimerComponent extends Component {
    public time_remaining: number;
}
```

Now we'll create an engine that checks the game score and emits a message if one player has reached that score.

```ts
import { Emits, Engine } from "encompass-ecs";
import { GoalOneComponent } from "game/components/goal_one";
import { GoalTwoComponent } from "game/components/goal_two";
import { ScoreComponent } from "game/components/score";
import { WinDisplayComponent } from "game/components/win_display";
import { WinMessage } from "game/messages/win";

@Emits(WinMessage)
export class CheckScoreEngine extends Engine {
    private winning_score: number;

    public initialize(winning_score: number) {
        this.winning_score = winning_score;
    }

    public update() {
        if (this.read_component(WinDisplayComponent)) { return; }

        const goal_one_component = this.read_component(GoalOneComponent);
        const goal_two_component = this.read_component(GoalTwoComponent);

        if (goal_one_component && goal_two_component) {
            const goal_one_entity = this.get_entity(goal_one_component.entity_id);
            const goal_two_entity = this.get_entity(goal_two_component.entity_id);

            if (goal_one_entity && goal_two_entity) {
                const score_one_component = goal_one_entity.get_component(ScoreComponent);
                const score_two_component = goal_two_entity.get_component(ScoreComponent);

                const score_one = score_one_component.score;
                const score_two = score_two_component.score;

                if (score_one >= this.winning_score) {
                    const win_message = this.emit_message(WinMessage);
                    win_message.player_index = 1;
                } else if (score_two >= this.winning_score) {
                    const win_message = this.emit_message(WinMessage);
                    win_message.player_index = 0;
                }
            }
        }
    }
}
```

Now we can create an Engine that receives the WinMessage and adds new components to the game world.

```ts
import { Engine, Reads } from "encompass-ecs";
import { FinishGameTimerComponent } from "game/components/finish_game_timer";
import { WinDisplayComponent } from "game/components/win_display";
import { WinMessage } from "game/messages/win";

@Reads(WinMessage)
export class WinEngine extends Engine {
    public update() {
        const win_messages = this.read_messages(WinMessage);

        for (const win_message of win_messages.values()) {
            const entity = this.create_entity();
            const component = entity.add_component(WinDisplayComponent);
            component.player_index = win_message.player_index;

            const timer_entity = this.create_entity();
            const timer_component = timer_entity.add_component(FinishGameTimerComponent);
            timer_component.time_remaining = 3;
        }
    }
}
```

We want to see a win message display, so let's create a new GeneralRenderer that shows one.

```ts
import { GeneralRenderer } from "encompass-ecs";
import { WinDisplayComponent } from "game/components/win_display";

export class WinRenderer extends GeneralRenderer {
    public layer = 2;

    private win_font: Font;
    private win_text: Text;

    private x: number;
    private y: number;
    private padding: number;

    public initialize(x: number, y: number, padding: number) {
        this.win_font = love.graphics.newFont("game/assets/fonts/Squared Display.ttf", 128);
        this.win_text = love.graphics.newText(this.win_font, "");

        this.x = x;
        this.y = y;
        this.padding = padding;
    }

    public render() {
        const win_component = this.read_component(WinDisplayComponent);

        if (win_component) {
            this.win_text.set("Player " + (win_component.player_index + 1) + " wins");

            const win_text_width = this.win_text.getWidth();
            const win_text_height = this.win_text.getHeight();

            love.graphics.setColor(0, 0, 0, 1);
            love.graphics.rectangle(
                "fill",
                this.x - win_text_width * 0.5 - this.padding,
                this.y - win_text_height * 0.5 - this.padding,
                win_text_width + this.padding,
                win_text_height + this.padding,
            );

            love.graphics.setColor(1, 1, 1, 1);
            love.graphics.rectangle(
                "line",
                this.x - win_text_width * 0.5 - this.padding,
                this.y - win_text_height * 0.5 - this.padding,
                win_text_width + this.padding,
                win_text_height + this.padding,
            );

            love.graphics.draw(
                this.win_text,
                this.x,
                this.y,
                0,
                1,
                1,
                this.win_text.getWidth() * 0.5,
                this.win_text.getHeight() * 0.5,
            );
        }
    }
}
```

Let's also make it so the ball stops spawning if the game is over. In our BallSpawnTimerEngine component loop:

```ts
...
    if (this.read_component(WinDisplayComponent)) {
        this.get_entity(component.entity_id)!.destroy();
        continue;
    }
...
```

Finally, we're going to need some logic to take us back to the title screen. We _could_ just pass the Game state to a FinishGameEngine... but then we would have a circular dependency. This is a prime candidate for an Interface.

Let's create a new folder, **game/interfaces** and a new file, **game/interfaces/finishable.ts**:

```ts
export interface IFinishable { finished: boolean; }
```

Now, in **game/states/game.ts**:

```ts
export class Game extends State implements IFinishable {
```

Now let's create our FinishGameEngine.

```ts
import { Engine, Mutates } from "encompass-ecs";
import { FinishGameTimerComponent } from "../components/finish_game_timer";
import { IFinishable } from "../interfaces/finishable";

@Mutates(FinishGameTimerComponent)
export class FinishGameEngine extends Engine {
    private game_state: IFinishable;

    public initialize(game_state: IFinishable) {
        this.game_state = game_state;
    }

    public update(dt: number) {
        const finish_game_timer_component = this.read_component_mutable(FinishGameTimerComponent);

        if (finish_game_timer_component) {
            finish_game_timer_component.time_remaining -= dt;

            if (finish_game_timer_component.time_remaining <= 0) {
                this.game_state.finished = true;
            }
        }
    }
}
```

This is pretty straightforward. Once the timer expires we change the game state to let the rest of the program know the state is finished. Now we can use this in our main loop. In **main.ts**:

```ts
love.update = (dt) => {
    if (current_state === title) {
        if (love.keyboard.isDown("space")) {
            current_state = game;
        }
    } else if (current_state === game) {
        if (game.finished) {
            game = new Game();
            game.load();

            current_state = title;
        }
    }
    current_state.update(dt);
};
```

Now when the game is finished, it creates a new Game and switches the current state back to the title menu.

Don't forget to add our new Engines to the WorldBuilder.

```ts
world_builder.add_engine(CheckScoreEngine).initialize(winning_score);
world_builder.add_engine(WinEngine);
world_builder.add_engine(FinishGameEngine).initialize(this);

world_builder.add_engine(WinRenderer).initialize(play_area_width * 0.5, play_area_height * 0.5, 20);
```

<video width="75%" autoplay="autoplay" muted="muted" loop="loop" style="display: block; margin: 0 auto;">
    <source src="/images/win.mp4" type="video/mp4">
</video>

That's it... we're done.
