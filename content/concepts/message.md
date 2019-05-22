---
title: "Message"
date: 2019-05-22T13:10:43-07:00
weight: 7
---

Similar to Components, Messages are collections of data.

Messages are used to transmit data between Engines so they can manipulate the game state accordingly.

To define a message, extend the Message class.

```ts
import { Message } from "encompass-ecs";

class MotionMessage extends Message {
    public x: number;
    public y: number;
}
```

Messages are temporary and destroyed at the end of the frame.

{{% notice notice %}}
Ok fine, since you asked, Messages actually live in an object pool so that they aren't garbage-collected at runtime. But you as the game developer don't have to worry about that.
{{% /notice %}}
