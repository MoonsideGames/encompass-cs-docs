---
title: "Component"
date: 2019-05-22T12:51:29-07:00
weight: 5
---

A Component is a collection of related data.

To define a Component, declare a struct which implements the **IComponent** interface.

```cs
using Encompass;
using System.Numerics;

public struct VelocityComponent : IComponent {
    public Vector2 velocity;
}
```

Components are attached to Entities with the **SetComponent** method.

```cs
using Encompass;

...

var worldBuilder = new WorldBuilder();
var entity = worldBuilder.CreateEntity();
worldBuilder.SetComponent(entity, new VelocityComponent { velocity = Vector2.Zero });
```

**SetComponent** can also be used from within an **Engine**. We will talk more about this later.

Components cannot exist apart from an Entity and are automagically destroyed when they are removed or their Entity is destroyed.

{{% notice warning %}}
Components should **never** reference other Components directly. This breaks the principle of loose coupling. You **will** regret it if you do this.
{{% /notice %}}
