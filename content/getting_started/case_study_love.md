---
title: "Case Study: LÖVE"
date: 2019-05-22T10:38:01-07:00
weight: 10
---

First, install [LÖVE](https://love2d.org).

Make sure you can run it from the terminal by running:

```sh
> love
```

You should see the LÖVE window pop up with the NO GAME screen. If you don't see this, check your terminal environment. On Windows you probably need to manually add the path where you installed LÖVE to your Path Environment Variable and then restart the machine. Thanks Windows!

Encompass-TS uses [Node.js](https://nodejs.org/) for its build process and [npm](https://www.npmjs.com/) to manage dependencies.
Install Node, which automatically installs npm.

Make sure it is properly installed by running:

```sh
> npm
```

You should see npm print out a bunch of help information. Now we're almost ready to begin.

Download the [Encompass/LÖVE starter project](https://github.com/encompass-ecs/encompass-love2d-starter). Place its contents in a folder and rename the folder to the name of your project. Change information in the `package.json` file where appropriate.

Now we are ready. Enter the project folder in your terminal and do:

```sh
> npm install
```

Encompass-TS will install everything it needs to compile your project to Lua.

The starter project contains some scripts to automate the build process.

To run your game, do:

```sh
> npm run love
```

Or, on Windows:

```sh
> npm run lovec
```

so that you get proper debug console output.

If you just want to build the game without running it, do:

```sh
> npm run build
```

That's everything you need to start making a game with Encompass and LÖVE! Wasn't that easy?
