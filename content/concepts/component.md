---
title: "Component"
date: 2019-05-22T12:51:29-07:00
weight: 5
---

A Component is a collection of related data.

To define a Component, extend the Component class.

```ts
import { Component } from "encompass-ecs";

class PositionComponent extends Component {
    public x: number;
    public y: number;
}
```

Components are created in context with an Entity.

```ts
const entity = World.create_entity();
const position = entity.add_component(PositionComponent);
position.x = 3;
position.y = -4;
```

Components cannot exist apart from an Entity and are automagically destroyed when they are removed or their Entity is destroyed.

{{% notice warning %}}
Components should **never** reference other Components. This breaks the principle of loose coupling. You **will** regret it if you do this.
{{% /notice %}}
