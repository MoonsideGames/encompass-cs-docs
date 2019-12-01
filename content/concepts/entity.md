---
title: "Entity"
date: 2019-05-22T12:55:22-07:00
weight: 10
---

An Entity is a structure composed of a unique ID and a collection of Components.

Entities do not have any implicit properties or behaviors. They are granted these by their collection of Components.

There is no limit to the amount of Components an Entity may have, but Entities may only have a single Component of a particular type.

Entities can also be destroyed, permanently removing them and their components from the World.

Entities are created either by the WorldBuilder or by Engines. (We'll get into these soon.)
