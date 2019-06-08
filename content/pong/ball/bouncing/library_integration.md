---
title: "Library Integration"
date: 2019-05-27T13:19:31-07:00
weight: 5
---

So we've found a library in our target language that implements some feature we want. In our case it's the [bump.lua](https://github.com/kikito/bump.lua) library that provides AABB collision detection.

Now we need to declare that library to TypeScript so we can use it in our game code.

If you'd like a very detailed description of declaration files and how they work, I recommend perusing the [official documentation](https://www.typescriptlang.org/docs/handbook/declaration-files/introduction.html). But I will give a specific use-case walkthrough here.

First, let's download _bump.lua_ and place it in the lua-lib folder. This ensures that the library will be included in your game during the build process.

In the lua-lib folder, let's create a new file: **lua-lib/bump.d.ts**

Now we need to find out which functions of the library we are actually going to use. If we look over the bump.lua documentation, we see that the library asks us to initialize a "world" using *bump.newWorld*, add rectangles to the world with *world:add*, and declare their movements using *world:move*. We can also use *world:check* and *world:update* for finer control over the rectangles.

First things first, the newWorld function.

```ts
export function newWorld(this: void, cell_size?: number): World;
```

{{% notice notice %}}
Why "this: void"?

In Lua, there are two ways to call a function: with the dot . syntax or the colon : syntax. The colon is used as shorthand for functions that need to have the `self` argument passed to the function being called.

Since _newWorld_ is called at the library level, "this: void" tells TypeScriptToLua that this function does not need `self` and should be called with the dot syntax.

For more information, check out the [TypeScriptToLua documentation](https://github.com/TypeScriptToLua/TypeScriptToLua/wiki/Functions-and-the-%60self%60-Parameter)
{{% /notice %}}

We'll need to declare the *World* interface as well, with its "add" and "move" functions.

```ts
export interface World {
    add(table: table, x: number, y: number, width: number, height: number): void;

    /** @tupleReturn */
    check(table: table, x: number, y: number, filter?: (item: table, other: table) => CollisionType): [number, number, Collision[], number];

    /** @tupleReturn */
    move(table: table, x: number, y: number): [number, number, Collision[], number];

    update(table: table, x: number, y: number, width?: number, height?: number): void;

    remove(table: table): void;
}
```

In Lua, *table* is our generic object. All of our classes and other objects are translated to tables when using TSTL, so it's a convenient type for generic objects.

"add" is a pretty straightforward function: it takes a position and a width and height and adds that rectangle to the World.

"check" and "move" are a bit more involved. See the return type there? That means that the function returns multiple variables. This is called a "tuple return" in Lua. _@tupleReturn_ tells the TSTL transpiler that the function returns a tuple so it can translate the call directly. We'll talk more about this in a second.

"check" and "move" return four variables. The first two are _actualX_ and _actualY_, which are the new positions after the move. The next is _cols_, which is a list of collisions detected during the move. The final is _len_, which is the total amount of collisions detected.

"remove" takes a table and removes it from the world. This is used when objects are destroyed and no longer exist in the game.

Inspecting the contents of _cols_ in the bump.lua documentation gives us the following types and interfaces:

```ts
export type CollisionType = "touch" | "cross" | "slide" | "bounce";

export interface Vector {
    x: number,
    y: number
}

export interface Rect {
    x: number,
    y: number,
    w: number,
    h: number
}

export interface Collision {
    item: table,
    other: table,
    type: CollisionType,
    overlaps: boolean,
    ti: number,
    move: Vector,
    normal: Vector,
    touch: Vector,
    itemRect: Rect,
    otherRect: Rect
}
```

Finally, "update" tells the World about the new position of an object. It's useful if we are using "check" instead of "move".

With that out of the way, we can integrate the collision detection functionality into our project.
