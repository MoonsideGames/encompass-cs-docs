---
title: "Center Line"
date: 2019-06-07T10:43:23-07:00
weight: 50
---

Now we need to draw the center line.

This will be a fairly basic GeneralRenderer - it doesn't need to react to anything.

```ts
import { GeneralRenderer } from "encompass-ecs";

export class CenterLineRenderer extends GeneralRenderer {
    public layer = 0;

    private middle: number;
    private height: number;

    public initialize(middle: number, height: number) {
        this.middle = middle;
        this.height = height;
    }

    public render() {
        love.graphics.setLineWidth(2);
        this.dotted_line(this.middle, 0, this.middle, this.height, 10, 10);
    }

    private dotted_line(x1: number, y1: number, x2: number, y2: number, dash: number, gap: number) {
        const dx = x2 - x1;
        const dy = y2 - y1;
        const angle = math.atan2(dy, dx);
        const st = dash + gap;
        const len = math.sqrt(dx * dx + dy * dy);
        const nm = (len - dash) / st;

        love.graphics.push();

        love.graphics.translate(x1, y1);
        love.graphics.rotate(angle);
        for (let i = 0; i < nm; i++) {
            love.graphics.line(i * st + gap * 0.5, 0, i * st + dash + gap * 0.5, 0);
        }

        love.graphics.pop();
    }
}
```

I took the dotted line draw procedure from [this helpful forum post](https://love2d.org/forums/viewtopic.php?t=83295) and modified it slightly. Thanks Ref!

Add it to the WorldBuilder...

```ts
world_builder.add_renderer(CenterLineRenderer).initialize(play_area_width * 0.5, play_area_height);
```

![center line](/images/center_line.png)
