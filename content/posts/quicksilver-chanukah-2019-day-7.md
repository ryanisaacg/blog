---
title: "A Quicksilver Chanukah, Day 7: Quicksilver's New Graphics"
date: 2019-12-28
toc: false
images:
tags:
  - quicksilver
  - quicksilver-chanukah-2019
  - quicksilver-2019
---

The graphics API of Quicksilver isn't as dire need of a rework as the [lifecycle APIs](../quicksilver-chanukah-2019-day-6), but I wanted to take the next breaking change as an opportunity to address some long-standing issues. In no particular order, they are:

## Low-Level Access

As it stands, Quicksilver talks to a very specific backend design for graphics. The API is internal-only and designed to wrap up all the GL code in one place, to keep the high-level drawing APIs platform independent. There's no way to expose the raw GL API directly to the user, as the backend talks directly to OpenGL 3.2 and WebGL 1. With `glow`, we could expose a cross-platform OpenGL context, but there are still differences between desktop and web GL. These problems, like different ways of describing the input and output in GLSL, are addressed by [`golem` (see announcement from 2 days ago.)](../quicksilver-chanukah-2019-day-5).

Providing users access to a lower-level graphics API also frees the high-level API from trying to provide custom shader support. A custom shader is best served with custom vertex inputs, which means custom functions to turn high-level draw commands into low-level vertex data. Instead of trying to solve this problem in the general case, Quicksilver can provide the mechanism to solve it yourself.

## Draw Order

The previous graphics API aggressively batches draws, including re-ordering `draw` commands to avoid changing state while rendering. If a `z` parameter (not included in the default, simple `draw`) is not provided, anything goes. Hypothetically this leads to faster rendering, but it also means that many users experience unexpected results with basic operations. The new API should be built on a simple draw-order guarantee: the first draw call is drawn furthest back, with each subsequent draw lying on top of the ones before it.

## Render-To-Texture

The `Surface` API supports render-to-texture, but it has a few problems. First, the API is unintuitive: rendering to a texture only works within a closure, for example. Also, it's plain buggy. `golem` provides a harder-to-misuse API for off-screen rendering, which can be used to build a nicer abstraction in Quicksilver itself.

## Screen Resizing and Projection

The current version of Quicksilver "magically" handles the window being resized, as well as projecting and unprojecting between world and screen coordinates. Often this is more trouble than it's worth: you want your UI at one projection and letterbox, and your game content at another. By exposing a few more knobs to the user (like the GL viewport), the `ResizeStrategy` API can be provided on top of other, orthogonal abstractions.

1. [The introduction post](../quicksilver-chanukah-2019)
2. [`platter`: An async file-loading API for desktop and web](../quicksilver-chanukah-2019-day-2)
3. [`gestalt`: An API to manage bundling and saving data locally on desktop and web](../quicksilver-chanukah-2019-day-3)
4. [`blinds`: An easy-to-use async wrapper of `winit`](../quicksilver-chanukah-2019-day-4)
5. [`golem`: An opinionated mostly-safe graphics library for desktop and web GL](../quicksilver-chanukah-2019-day-5)
6. [Changes to the Quicksilver application lifecycle](../quicksilver-chanukah-2019-day-6)
7. This post!
8. An overview of the work on web support this year
