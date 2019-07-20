---
title: The State of Quicksilver Feb 2019
date: 2019-02-04
draft: false
toc: false
images:
tags:
  - quicksilver
  - rust
  - gamedev
---

## What is Quicksilver?

For anyone who doesn't know, [Quicksilver](https://github.com/ryanisaacg/quicksilver) is my 2D game framework that targets desktop and web. It is a pure-Rust library that focuses on ergonomics and simplicity.

## The State of Rust (WASM) Game Development

Quicksilver is part of a small (but growing) set of Rust game engines. Notable are the Amethyst project, an open-source game engine, and ggez, a LOVE-inspired 2D game framework.
Increasingly it is practical to build an entire game with only the Rust toolchain: winit provides windowing, a variety of crates allow access to platform graphics APIs, rodio provides sound, rusttype for font rendering, etc. A pure-Rust game framework is getting easier to make all the time.

However, very few libraries support WASM out of the box. This means Quicksilver and Rust WASM games in general essentially have small implementations of windowing, event handling, sounds, graphics, and input for the web that have not been contributed upstream.

## The Framework Problem

Opting to use Quicksilver for some feature (web support, API design, automatic batching) requires opting into *all* of Quicksilver's features. This isn't necessarily great! Even with compile-time feature flags available, it's hard to do any Quicksilver code without buying into the State trait. This boils down to the vision of Quicksilver as a framework: if you can just plug in Quicksilver and start making a game right away, it needs to be everything to everyone.

Partly this is due to Quicksilver's history as a bit of a learning project for me, which I wanted to build from scratch in as many places as possible. It's also caused by my impatience: the Rust ecosystem is amazing and featureful, but waiting for every single upstream project to accept changes would mean that Quicksilver could only be developed as fast as its slowest dependency. I tend to have time to work on the project in bursts, when I clear out the bug backlog and implement a few features from the roadmap; this doesn't mesh well with long-running interactions with other projects. The way I've handled this in the past is writing my own Javascript bindings for each feature, essentially cloning an entire crate.

## The Fundamentals

Largely, Quicksilver's API is approaching a state of stability. It accomplishes most of what I want it to, in a way I'm largely satisfied with. For end users, the Quicksilver project has been more or less stable for a few months, and no substantial breaking changes are planned for the future.

However, Quicksilver still has quite a few systems implemented by a dependency and an ad-hoc web clone of that dependency. This is what I hope to change, by contributing web backends across the ecosystem. My ideal outcome is that the fundamental multimedia crates (winit, rodio, gilrs, etc.) can run on WASM out-of-the-box.

## Going Forward

If it sounds like my current goal is to make Quicksilver obsolete, that wouldn't be too far off. Quicksilver's raison d'etre is packaging up a bundle of functionality and to provide a single API for both web and native. Moving the ecosystem to support web and native makes this less important. Parts of Quicksilver that I think are particularly useful will probably become their own crates (re-exported in Quicksilver) to allow their use outside the framework. Some future version of Quicksilver may just be entirely a re-export of various crates and types with a consistent organization.

In essence, I want seamless desktop and WASM development to move from a selling point of developing games in Quicksilver to a selling point of developing games in Rust.

