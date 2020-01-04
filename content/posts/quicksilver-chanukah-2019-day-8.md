---
title: "A Quicksilver Chanukah, Day 8: Rust Gamedev and Web"
date: 2019-12-29
toc: false
images:
tags:
  - quicksilver
  - quicksilver-chanukah-2019
  - quicksilver-2019
  - winit
---

This year has been great for Rust gamedev on the web. Huge progress has been made towards a full game stack being available more-or-less "for free:" most of the foundational crates have available web support, in one way or another.

## Windowing and Input

For opening a window and receiving events from it, the Rust ecosystem has [`winit`](https://github.com/rust-windowing/winit). Along with [Héctor Ramón (hecrj)](https://github.com/hecrj) and and [Ben Merritt (blm768)](https://github.com/blm768), I worked on adding web support. [It has been merged and released](https://users.rust-lang.org/t/winit-0-20-and-web-support/36155), and is used in [coffee](https://github.com/hecrj/coffee) and [blinds](https://github.com/ryanisaacg/blinds). While using `winit` directly does require some glue code to plug your canvas into the webpage, intermediate layers like game frameworks can take care of the details.

While `winit` doesn't currently provide gamepad events, [`gilrs`](https://gitlab.com/gilrs-project/gilrs) fills that gap nicely. In the beginning of the year I added web support via stdweb, which should translate to `wasm-bindgen` when necessary. Unlike `winit`, which requires a little glue code to connect to your webpage, `gilrs` Just Works. It functions nearly the same as the desktop version, with the only limitations being those the web platform imposes.

## Graphics

When it comes to graphics on the web, there are two APIs to keep in mind. One is WebGL, which is the current way of writing graphics code for the web. The other is WebGPU, an evolving standard based on newer, more modern APIs that will be the way forward. Currently WebGPU is just in the draft phase, so we can't target it just yet. Targeting WebGL directly is an option, but it seems like a waste to write OpenGL code for desktop and then again for web. Enter [`glow`](https://github.com/grovesNL/glow), which brings a unified API across destop and WebGL.

Built on top of `glow` is `gfx-backend-gl`, which brings [`gfx-hal`](https://github.com/gfx-rs/gfx) to desktop GL and web. By extension comes [`wgpu`](https://github.com/gfx-rs/wgpu-rs), an idiomatic Rust implementation of the upcoming WebGPU specification. It uses WebGL for now, but when WebGPU is stabilization and available, we'll have a modern graphics API that works seamlessly across desktop and web.

## Audio

The audio story for Rust on the web is not there yet, unfortunately. [`cpal`](https://github.com/RustAudio/cpal), a cross-platform library for Rust audio, doesn't have web support yet. However, there is hope! [Mozilla announced a grant to Nannou](https://nannou.cc/posts/moss_grant_announce) that includes web support in `cpal` as a core goal. The relevant issue is [cpal #212](https://github.com/RustAudio/cpal/issues/212), which will hopefully yield some information early next year.


That's it for my updates! This was a good year for Rust on the web, and laid the groundwork for the changes I've talked about in Quicksilver. I'll be back in the new year with a State of Quicksilver 2020, which should come along with an alpha release of the new Quicksilver version.

If you're interested in updates from other places in the Rust ecosystem, check out the [State of GGEZ 2020](https://wiki.alopex.li/TheStateOfGGEZ2020) and the [Rust Gamedev Working Group's newsletters](https://rust-gamedev.github.io/). See you in the new year!

1. [The introduction post](quicksilver-chanukah-2019)
2. [`platter`: An async file-loading API for desktop and web](quicksilver-chanukah-2019-day-2)
3. [`gestalt`: An API to manage bundling and saving data locally on desktop and web](quicksilver-chanukah-2019-day-3)
4. [`blinds`: An easy-to-use async wrapper of `winit`](quicksilver-chanukah-2019-day-4)
5. [`golem`: An opinionated mostly-safe graphics library for desktop and web GL](quicksilver-chanukah-2019-day-5)
6. [Changes to the Quicksilver application lifecycle](quicksilver-chanukah-2019-day-6)
7. [Changes to the Quicksilver graphics API](../quicksilver-chanukah-2019-day-7)
8. This post!
