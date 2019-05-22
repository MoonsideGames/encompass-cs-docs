---
title: "Engine"
date: 2019-05-22T13:01:28-07:00
weight: 15
---

An Engine is the Encompass notion of an ECS System. Much like the engine on a train, your Engines make the simulation move along.

{{% notice notice %}}
I never liked the term System. It is typically used to mean structures in game design and I found this confusing when discussing code implementation vs design.
{{% /notice %}}

Engines are responsible for reading the game state, reading messages, emitting messages, and creating or mutating Entities and Components.

An Engine which Reads a particular message is guaranteed to run *after* all Engines which Emit that particular message.

To define an Engine, extend the Engine class.

Here is an example Engine:

```ts
import { Engine, Mutates, Reads } from "encompass-ecs";
import { LogoUIComponent } from "../../components/ui/logo";
import { ShowUIMessage } from "../../messages/show_ui";

@Reads(ShowUIMessage)
@Mutates(LogoUIComponent)
export class LogoDisplayEngine extends Engine {
  public update() {
    const logo_ui_component = this.read_component_mutable(LogoUIComponent);
    if (logo_ui_component && this.some(ShowUIMessage)) {
      logo_ui_component.image.isVisible = true;
    }
  }
}
```

If a LogoUIComponent exists, and a ShowUIMessage is received, it will make the logo image on the LogoUIComponent visible. Simple!
