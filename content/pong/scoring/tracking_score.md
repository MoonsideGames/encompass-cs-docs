---
title: "Tracking the Score"
date: 2019-06-04T16:49:50-07:00
weight: 30
---

Finally, we need to track the score and update it appropriately.

I think a scoring Component is appropriate.

```ts
import { Component } from "encompass-ecs";

export class ScoreComponent extends Component {
    public score: number;
}
```

We should have two different scores, and they should update based on which specific goal is touched by the ball.

I think we need another tag component.

```ts
import { Component } from "encompass-ecs";

export class GoalOneComponent extends Component {}
```

```ts

import { Component } from "encompass-ecs";

export class GoalTwoComponent extends Component {}
```

I know what you're thinking. Why not just have one GoalComponent with an index on it? Encompass lets us retrieve components from the game state by type, and it does so very quickly, so it is good to create structures that let us use that feature. Consider:

```ts
const goal_one_component = this.read_component(GoalOneComponent);
```

vs.

```ts
const goal_components = this.read_components(GoalComponent);
let goal_component;
for (const component of goal_components.values()) {
    if (component.goal_index === 1) {
        goal_component = component;
        break;
    }
}
```

The second one is way worse to read right? It's also much slower, performance-wise. Make use of Encompass's component retrieval structure wherever you can.

You know the drill by now.

Let's add a new property to our GoalSpawnMessage so we can tell which one is which.

```ts
public goal_index: number;
```

Let's add this in our GoalSpawner:

```ts
const score_component = entity.add_component(ScoreComponent);
score_component.score = 0;

if (message.goal_index === 0) {
    entity.add_component(GoalOneComponent);
} else if (message.goal_index === 1) {
    entity.add_component(GoalTwoComponent);
}
```

Now, we can tell which goal needs to have its score updated.

Let's create a score update message.

In **game/messages/score.ts**:

```ts
import { ComponentMessage, Message } from "encompass-ecs";
import { ScoreComponent } from "game/components/score";

export class ScoreMessage extends Message implements ComponentMessage {
    public component: Readonly<ScoreComponent>;
    public delta: number;
}
```

Now we can update our BallGoalCollisionEngine.

```ts
const score_component = message.goal_entity.get_component(ScoreComponent);
const score_message = this.emit_component_message(ScoreMessage, score_component);
score_message.delta = 1;
```

And let's create an engine to update the score.

In **game/engines/score.ts**:

```ts
import { Engine, Mutates, Reads } from "encompass-ecs";
import { ScoreComponent } from "game/components/score";
import { ScoreMessage } from "game/messages/score";

@Reads(ScoreMessage)
@Mutates(ScoreComponent)
export class ScoreEngine extends Engine {
    public update() {
        for (const score_message of this.read_messages(ScoreMessage).values()) {
            const score_component = this.make_mutable(score_message.component);
            score_component.score += score_message.delta;
        }
    }
}
```

And add it to our WorldBuilder:

```ts
world_builder.add_engine(ScoreEngine);
```

Last, but not least, it would be nice to actually see the score being drawn on the screen.
