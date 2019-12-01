---
title: "Message"
date: 2019-05-22T13:10:43-07:00
weight: 7
---

Similar to Components, Messages are collections of data.

Messages are used to transmit data between Engines so they can manipulate the game state accordingly.

To define a message, declare a struct which implements the IMessage interface.

```cs
using Encompass;

public struct MotionMessage : IMessage {
    public Vector2 motion;
}
```

Messages are temporary and destroyed at the end of the frame.

{{% notice notice %}}
Because structs are value types, we can create as many of them as we want without worrying about creating pressure on the garbage collector. Neato!
{{% /notice %}}
