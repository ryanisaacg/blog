---
title: "A Quicksilver Chanukah, Day 6: Quicksilver's New Lifecycle"
date: 2019-12-27
toc: false
images:
tags:
  - quicksilver
  - quicksilver-chanukah-2019
  - quicksilver-2019
---

Quicksilver has always had some major API compromises to deliver on its core promise: write a game once, and it targets desktop and the web with no changes. The two ugliest compromises are the `State` and `Asset` APIs, for managing your application's core loop and file loading, respectively.

`State` is a trait that Quicksilver uses to manage when your code runs. It requires you define the `new` method, as well as `draw`. Optionally you can define `update`, for a fixed tick rate function, and `event` for handling individual input events. This type then gets passed as a generic parameter to the `run` function, which handles instantiating it and running the event loop. The downside is that the user gets very little say in how their code is arranged, but the benefit is that your code runs on web without blocking the main thread (which will lock up the tab.)

`Asset` is a wrapper around `Future`, which is an asychronous action in Rust. The point of the `Asset` API was to avoid users having to manually poll their futures to check if they're ready: just use an `Asset` and its execute method, which will run a closure if its `Future` is completed. Unfortunately, this turns out not to be very ergonomic. `Asset` is a pain to use, and a point of confusion for most users. There's no way around asychronous file loading on the web, but new developments in Rust have made much better solutions possible.

Readers of previous posts may notice that these two API problems are addressed by two of the crates I announced this week: `blinds` to address `State` and `platter` to address `Asset`. Where before, you might write something like:

```rust
struct MyState {
    images: Asset<Vec<Image>>,
    game: Game,
}

impl State for MyState {
    fn new() -> MyState {
        MyState {
            images: Asset::new(join_all(vec![Image::load("a"), Image::load("b"), Image::load("c")])),
            game: Game::new()
        }
    }

    fn event(&mut self, _win: &mut Window, ev: &Event) {
        self.images.execute(|images| {
            self.game.process_event(ev);
        });
    }

    fn update(&mut self, _win: &mut Window) {
        self.images.execute(|images| {
            self.game.update();
        });
    }

    fn draw(&mut self, win: &mut Window) {
        self.images.execute(|images| {
            self.game.draw(&images, win);
        });
    }
}

fn main() {
    run::<MyState>();
}
```

you could now write something like:

```rust
async fn my_game(win: Window, mut gfx: Graphics, events: EventStream) {
    let mut game = Game::new();
    let images = join_all(vec![Image::load("a"), Image::load("b"), Image::load("c")]).await;
    loop {
        while let Some(ev) = events.next_event().await {
            game.process_event(ev);
        }
        game.update();
        game.draw(&mut gfx);
        gfx.present(&win);
    }
}

fn main() {
    run(my_game);
}
```

If you want to know more, check out [my post about blinds](../quicksilver-chanukah-2019-day-4) for more details on the event loop and [my post about platter](../quicksilver-chanukah-2019-day-2) for more details on async file loading.

As a side effect of the async API, the user has a lot more control over when their code runs. Previously Quicksilver managed your update timing automatically: it would aim for 60Hz by default, regardless of how long updates take. Now you're in charge of your own timing, though features to make timing fire-and-forget could become part of the final release.

Speaking of releases, I hope to have a very early alpha of this version of Quicksilver out within the next week or so. It will be a very bare-bones version: just the event loop and some small graphics functions for now, which I'll go into more depth about tomorrow.

1. [The introduction post](../quicksilver-chanukah-2019)
2. [`platter`: An async file-loading API for desktop and web](../quicksilver-chanukah-2019-day-2)
3. [`gestalt`: An API to manage bundling and saving data locally on desktop and web](../quicksilver-chanukah-2019-day-3)
4. [`blinds`: An easy-to-use async wrapper of `winit`](../quicksilver-chanukah-2019-day-4)
5. [`golem`: An opinionated mostly-safe graphics library for desktop and web GL](../quicksilver-chanukah-2019-day-5)
6. This post!
7. [Changes to the Quicksilver graphics API](../quicksilver-chanukah-2019-day-7)
8. [An overview of the work on web support this year](../quicksilver-chanukah-2019-day-8)
