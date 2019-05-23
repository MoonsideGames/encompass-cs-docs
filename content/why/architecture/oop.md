---
title: "OOP"
date: 2019-05-21T15:54:18-07:00
weight: 20
---

##### *They call it OOP because it was a mistake. --Unknown*

You are probably very familiar with OOP, or object-oriented programming, as a game designer. It is the structural idea behind most games as they are written today, though this is slowly changing.

OOP is a structure based on the concept of **Objects**. Objects contain both data, referred to as *properties*, and logic, referred to as *methods*.

Object orientation is an intuitive idea when it comes to building simulation-oriented applications such as video games. We think of each "thing" in the game as a self-contained object which can be acted upon externally via methods. For example, in the game Asteroids, we could think of the game this way: the ship is an object, the bullets the ship fires are objects, the asteroids are objects, and so on.

Unfortunately, things aren't quite this simple when it comes to more complex games.

As programmers we want to re-use code as much as possible. Every bit of duplication is an opportunity for bugs to lurk in our program. Object-oriented code accomplishes re-use with a concept called *inheritance*. With inheritance, classes can be partially based on other classes. Maybe a Ball class has a position and a velocity and a bounciness property. A BouncyBall would inherit from Ball and have a greater value in its bounciness property. Simple enough, right?

But we soon run into problems. In game development we often wish to mix-and-match behaviors. Suppose I have an object where it would make sense to inherit from *two* different classes. Now... we are hosed! Why? If two parent classes have a property or a method with the same name, now our child object has no idea what to do. Most object-oriented systems, in fact, forbid multiple inheritance, and the ones that don't forbid it require very complex definitions to make it work. So we end up having to share code via helper functions, or giant manager classes, or other awkward patterns.

We also run into an issue called *tight coupling*. Objects that reference each other's properties or methods directly become a problem when we change the structure of those objects in any way. If we modify the structure of object B, and object A references object B, then we have to also modify object A. In a particularly poorly structured system, we might have to modify a dozen objects just to make a slight modification to the behavior of a single object.

Tight coupling is our worst nightmare as game programmers. Games are, by nature, extremely complex simulations. The more coupling we have between objects, the more of the entire environment of the game we have to understand before we can pick apart the specific behavior of a single object, let alone the whole game itself. It is very possible that we can surprise ourselves by unexpectedly changing the behavior of different objects when modifying a single object. And we hate surprises.

We want our architecture to encourage us to create as little coupling as possible between different elements of the game, so that we can look at any individual element of the game and easily understand how it behaves without needing to deeply understand the other elements of the game. We also wish to modify behavior without introducing unexpected changes in other seemingly unrelated objects. OOP works in very specific encapsulated cases, but it is not a satisfactory structure for implementing game behavior.

It turns out there is an architecture that addresses our desire to be able to compose objects out of several different behaviors. Let's learn about it.
