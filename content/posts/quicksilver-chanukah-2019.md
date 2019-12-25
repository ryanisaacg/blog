---
title: "A Quicksilver Chanukah: Day 1"
date: 2019-12-22
toc: false
images:
tags:
  - quicksilver
  - quicksilver-chanukah-2019
  - quicksilver-2019
---

I started this year with a [blog post about the state of Quicksilver](../state-of-quicksilver-feb-2019) where I set a goal for myself: get web support for important game development crates upstreamed. At the end of the year, I'm happy to update this work (by me and many others!) has been a success, and the core of Rust's game ecosystem is now-web enabled. In the future, I want to write a brief overview of those changes, but for now I want to focus on their relevance to Quicksilver.

Without having to maintain my own set of web bindings for Quicksilver, I was able to start working on significant changes to both the internal and external structure. There are a few entirely unrelated modules in Quicksilver that are all needed for the framework: handling filesystem resources, creating and loading local data, the event loop, the graphics API, and the sound API. It makes sense to allow users to pick and choose these systems, which is the goal of Quicksilver's optional features. However, that still requires depending on Quicksilver, which doesn't make sense for some projects. Instead, I've created a few new crates that anyone can drop into their project without pulling in any other Quicksilver-related code.

Additionally, a major new Rust feature landed this year: async/await. Quicksilver's asset loading system has been a sticking point, because of its design decisions and the inherent pain of combinator-based `Future` code. With `async` and `.await`, a new asynchronous asset loading API can be almost seamless. Combined with fixes to long-standing graphics API problems, I've been working on a new major release for Quicksilver (which will be the first breaking change in over a year.)

As [Chanukah](https://en.wikipedia.org/wiki/Hanukkah) starts tonight, I'm going to share these new crates and big Quicksilver updates over the next eight days. I can't promise I'll actually stick to this schedule, but my goal is:

1. This post!
2. [`platter`: An async file-loading API for desktop and web](../quicksilver-chanukah-2019-day-2)
3. [`gestalt`: An API to manage bundling and saving data locally on desktop and web](../quicksilver-chanukah-2019-day-3)
4. `blinds`: An easy-to-use async wrapper of `winit`
5. `golem`: An opinionated mostly-safe graphics library for desktop and web GL
6. Changes to the Quicksilver application lifecycle
7. Changes to the Quicskilver graphics API
8. An overview of the work on web support this year
