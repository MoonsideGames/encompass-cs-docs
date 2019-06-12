---
title: "Drawing the Score"
date: 2019-06-04T17:21:52-07:00
weight: 40
---

Remember Renderers? Haven't thought about those in a while.

All we need to draw new elements to the screen are Renderers. Let's create a new GeneralRenderer.

But first, we're gonna need a font. I liked [this font](https://www.dafont.com/squared-display.font). But you can pick any font you like. It's your world and you can do whatever you like in it.

Place the font of your heart's desire into the directory **game/assets/fonts**. Then it can be used in your game.

Let's write our ScoreRenderer.

In **game/renderers/score.ts**:

```ts
import { Component, GeneralRenderer, Type } from "encompass-ecs";
import { GoalOneComponent } from "game/components/goal_one";
import { GoalTwoComponent } from "game/components/goal_two";
import { ScoreComponent } from "game/components/score";

export class ScoreRenderer extends GeneralRenderer {
    public layer = 1;

    private midpoint: number;
    private score_font: Font;
    private player_one_score_text: Text;
    private player_two_score_text: Text;

    public initialize(midpoint: number) {
        this.midpoint = midpoint;
        this.score_font = love.graphics.newFont("game/assets/fonts/Squared Display.ttf", 128);
        this.player_one_score_text = love.graphics.newText(this.score_font, "0");
        this.player_two_score_text = love.graphics.newText(this.score_font, "0");
    }

    public render() {
        this.render_score(GoalTwoComponent, this.player_two_score_text, this.midpoint - 200, 30);
        this.render_score(GoalOneComponent, this.player_one_score_text, this.midpoint + 200, 30);
    }

    private render_score(ComponentType: Type<Component>, score_text: Text, x: number, y: number) {
        const goal_component = this.read_component(ComponentType);
        if (goal_component) {
            const entity = this.get_entity(goal_component.entity_id);
            if (entity) {
                const score_component = entity.get_component(ScoreComponent);
                if (score_component) {
                    score_text.set(score_component.score.toString());

                    love.graphics.draw(
                        score_text,
                        x,
                        y,
                        0,
                        1,
                        1,
                        score_text.getWidth() * 0.5,
                        score_text.getHeight() * 0.5,
                    );
                }
            }
        }
    }
}
```

Basically, we find each goal component, grab its score component, and draw the score component's value to the screen as text.

If we create new LOVE Text object every frame, this is very performance heavy. So we want to create a Text on initialization and then set its contents instead.

It's also very expensive to create a new LOVE Font every frame. Like the Text objects, we store it on the Renderer.

Let's add our ScoreRenderer to the WorldBuilder.

```ts
world_builder.add_renderer(ScoreRenderer).initialize(play_area_width * 0.5);
```

<video width="75%" autoplay="autoplay" muted="muted" loop="loop" style="display: block; margin: 0 auto;">
    <source src="/images/score.webm" type="video/webm">
</video>

Look at that! It's starting to look and feel like a more complete game now.
