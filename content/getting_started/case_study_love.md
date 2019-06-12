---
title: "Case Study: LÖVE"
date: 2019-05-22T10:38:01-07:00
weight: 12
---

LÖVE is a 2D game engine that uses Lua as its scripting language. Because of this, we can use [Encompass-TS](https://github.com/encompass-ecs/encompass-ts) with the [TypescriptToLua transpiler](https://github.com/TypeScriptToLua/TypeScriptToLua) to make games for it.

If you are new to TypeScript, or even new to programming in general, I _strongly_ recommend reviewing the [TypeScript handbook](https://www.typescriptlang.org/docs/handbook/basic-types.html).

Install [LÖVE](https://love2d.org).

Make sure you can run it from the terminal by running:

```sh
love
```

You should see the LÖVE window pop up with the NO GAME screen. If you don't see this, check your terminal environment. On Windows you probably need to manually add the path where you installed LÖVE to your Path Environment Variable and then restart the machine. Thanks Windows!

Encompass-TS uses [Node.js](https://nodejs.org/) for its build process and [npm](https://www.npmjs.com/) to manage dependencies.
Install Node, which automatically installs npm.

Make sure it is properly installed by running:

```sh
npm
```

You should see npm print out a bunch of help information. Now we're almost ready to begin.

Download the [Encompass/LÖVE starter project](https://github.com/encompass-ecs/encompass-love2d-starter). Place its contents in a folder and rename the folder to the name of your project. Change information in the `package.json` file where appropriate.

Now we are ready. Enter the project folder in your terminal and do:

```sh
npm install
```

This will install everything you need to compile your project to Lua.

The starter project contains some scripts to automate the build process.

To create a clean build from scratch, do:

```sh
npm run build
```

To incrementally build your game as you make changes, do:

```sh
npm run watch
```

This will run continuously and build only files that change. It will reduce your build times significantly, so I recommend using it!

To copy your assets into the game build, do:

```sh
npm run copyassets
```

Make sure to do this if you are in incremental mode and make changes to assets.

To run your game, do:

```sh
npm run love
```

Or, on Windows:

```sh
npm run lovec
```

so that you get proper debug console output.

If you want to completely clean your build folder without making a new build, do:

```sh
npm run clean
```

That's everything you need to start making a game with Encompass and LÖVE!
