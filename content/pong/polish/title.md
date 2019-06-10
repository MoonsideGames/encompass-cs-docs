---
title: "Title"
date: 2019-06-09T16:51:13-07:00
weight: 20
---

It would be nice to have a title screen. Let's make that happen.

I would like us to have a concept of game state. The title menu is a pretty distinct thing from the game itself so it feels nicer to have it be self contained instead of managing extra state in the game world.

Let's create a new class in **game/state.ts**:

```ts
export abstract class State {
    public abstract load(): void;
    public abstract update(dt: number): void;
    public abstract draw(): void;
}
```

Remember, _abstract_ means that the class cannot be used directly, but describes features that exist in inherited classes. So we know that anything we make that inherits from State must have a load(), update(dt), and draw() method.

Let's create a new folder, **game/states**, and put **game.ts** in there. Let's also make it inherit from State:

```ts
export class Game extends State {
```

Let's make a new State called Title. It doesn't need to do much - just display the game title and a prompt for the player to start the game.

```ts
import { State } from "game/state";

export class Title extends State {
    private title_font: Font;
    private title_text: Text;

    private play_font: Font;
    private play_text: Text;

    public load() {
        this.title_font = love.graphics.newFont("game/assets/fonts/Squared Display.ttf", 128);
        this.title_text = love.graphics.newText(this.title_font, "Encompass Pong");

        this.play_font = love.graphics.newFont("game/assets/fonts/Squared Display.ttf", 32);
        this.play_text = love.graphics.newText(this.play_font, "Press Space");
    }

    public update() {}

    public draw() {
        love.graphics.draw(
            this.title_text,
            640,
            240,
            0,
            1,
            1,
            this.title_text.getWidth() * 0.5,
            this.title_text.getHeight() * 0.5,
        );

        love.graphics.draw(
            this.play_text,
            640,
            480,
            0,
            1,
            1,
            this.play_text.getWidth() * 0.5,
            this.play_text.getHeight() * 0.5,
        );
    }
}
```

Now in **main.ts** we can put code to handle our states.

```ts
let menu: Title;
let game: Game;
let current_state: State;

love.load = () => {
    ...

    menu = new Menu();
    menu.load();

    game = new Game();
    game.load();

    current_state = menu;
};

love.update = (dt) => {
    current_state.update(dt);
    if (current_state === menu) {
        if (love.keyboard.isDown("space")) {
            current_state = game;
        }
    }
};

love.draw = () => {
    current_state.draw();

    ...
}
```

The final result of **main.ts** should look like this.

```ts
declare global {let PROF_CAPTURE: boolean; }
PROF_CAPTURE = false; // set this to true to enable profiling

import * as jprof from "encompass-jprof";
import { State } from "game/state";
import { Game } from "game/states/game";
import { Title } from "game/states/title";

let menu: Title;
let game: Game;
let current_state: State;

love.load = () => {
    love.window.setMode(1280, 720, {vsync: false, msaa: 2});
    love.math.setRandomSeed(os.time());
    love.mouse.setVisible(false);

    menu = new Title();
    menu.load();

    game = new Game();
    game.load();

    current_state = menu;
};

love.update = (dt) => {
    current_state.update(dt);
    if (current_state === menu) {
        if (love.keyboard.isDown("space")) {
            current_state = game;
        }
    }
};

love.draw = () => {
    current_state.draw();

    love.graphics.setBlendMode("alpha");
    love.graphics.setColor(1, 1, 1, 1);
    love.graphics.print("Current FPS: " + tostring(love.timer.getFPS()), 10, 10);
};

love.quit = () => {
    jprof.write("prof.mpack");
    return false;
};
```

Let's try it!

![pong title](/images/pong-title.png)

Nice!
