---
title: "A Quicksilver Chanukah, Day 4: Blinds"
date: 2019-12-25
toc: false
images:
tags:
  - quicksilver
  - quicksilver-chanukah-2019
  - quicksilver-2019
  - blinds
---

With the recent update to [`winit`](https://github.com/rust-windowing/winit), it gained web support! This is great, and it means that Quicksilver doesn't need to have its own parallel implementation of windowing. However, there is room for an abstraction that's a little simpler than Winit, and takes advantage of `async/.await`, which is where [`blinds`](https://crates.io/crates/blinds) comes in.

`blinds` is based on `winit`, but focused on Quicksilver's specific use case of single-window games on desktop and web. It also integrates `gilrs` for gamepad support (though this could be provided directly by `winit` later), without exposing either in its external API. Additionally, `blinds` has optional support for OpenGL, via `glow` for uniform bindings across web and desktop.

The `async` API works via the `LocalPool` and the `winit` event loop, allowing you to write code that *almost* looks like a synchronous event loop.

```rust
use blinds::{run, Event, EventStream, Key, Settings, Window};

run(Settings::default(), app);

async fn app(_window: Window, mut events: EventStream) {
    loop {
        while let Some(ev) = events.next_event().await {
            // Process your events here!
        }
        // Process your frame here!
    }
}
```

The use of `.await` here is a bit of a hack, as it allows the event loop to wrest control away from the user and return it to the browser without blocking. Compare this to the comparable snippet from the legacy Quicksilver API:

```rust
use quicksilver::{
    Result,
    geom::Vector,
    lifecycle::{Event, Settings, State, Window, run},
};

struct HelloWorld;

impl State for HelloWorld {
    fn new() -> Result<HelloWorld> {
        Ok(HelloWorld)
    }
    
    fn event(&mut self, window: &mut Window, event: &Event) -> Result<()> {
        // Process events here
        Ok(())
    }
    
    fn update(&mut self, window: &mut Window) -> Result<()> {
        // Update here
        Ok(())
    }

    fn draw(&mut self, window: &mut Window) -> Result<()> {
        // Draw here
        Ok(())
    }
}

fn main() {
    run::<HelloWorld>("Hello World", Vector::new(800, 600), Settings::default());
}
```

The `async` API isn't just way more convenient than the previous, trait-based system. It also allows you to use other `async` APIs (like [day 2's `platter`](../quicksilver-chanukah-2019-day-2)), which is important for web support.

`blinds` isn't quite ready for a full release yet: there are a few more window APIs I do want to expose before it leaves alpha. However, you can still try it out, just be aware that the API is fairly unstable and likely to change between multiple times before the first non-alpha release!

With `blinds` today, tomorrow will be the last new crate: `golem`!

1. [The introduction post](../quicksilver-chanukah-2019)
2. [`platter`: An async file-loading API for desktop and web](../quicksilver-chanukah-2019-day-2)
3. [`gestalt`: An API to manage bundling and saving data locally on desktop and web](../quicksilver-chanukah-2019-day-3)
4. This post!
5. `golem`: An opinionated mostly-safe graphics library for desktop and web GL
6. Changes to the Quicksilver application lifecycle
7. Changes to the Quicskilver graphics API
8. An overview of the work on web support this year
