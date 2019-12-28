---
title: "A Quicksilver Chanukah, Day 2: Platter"
date: 2019-12-23
toc: false
images:
tags:
  - quicksilver
  - quicksilver-chanukah-2019
  - quicksilver-2019
  - platter
---

This year saw the release of `async/.await` in stable Rust, marking a huge change in how asynchronous code is written. Previously, one would have to chain a series of combinators onto a `Future` instance to accomplish asyncrhonous tasks. In Quicksilver, this was used for asset loading (for compatibility with the web backend), and has been a frequent source of pain.

Something as simple as loading a file, reading its contents, and loading a series of images based on those contents might look like this psuedo-Rust:

```rust
let images = load_file("my_manifest_path")
    .map(parse_file_contents)
    .and_then(|image_paths| join_all(
        image_paths
            .map(load_file)
            .map(|file| file.and_then(parse_image))
    );

render_loading();
loop {
    // Core application loop
    if images.is_ready {
        render_frame(&images);
    }
}
```

This is fairly hard to read, and requires a bit of domain-specific knowledge of the combinators to achieve it. Worse still is introducing more than one of these combinator chains, and trying to execute game logic only if they complete. This snippet is plenty complex, and hasn't even touched error handling.

Introducing [`platter`](https://crates.io/crates/platter), a new async-enabled crate with a single, simple task: load files on desktop and web with an easy API.

```rust
render_loading();
let contents = load_file("my_manifest_path").await?;
let image_paths = parse_file_contents(contents);
let images = Vec::new();
for image_path in image_paths {
    let image_file = load_file(image_path).await?;
    images.push(parse_image(image_file)?);
}
loop {
    render_frame(&images);
}
```

We can still make use of the combinators in a more functional-programming style if we wish, but now it's much clearer:

```rust
render_loading();
let contents = load_file("my_manifest_path").await;
let image_futures = parse_file_contents(contents)
    .map(load_file);
let images: Vec<Image> = try_join_all(image_futures)
    .await?
    .map(parse_image)
    .collect()?;
loop {
    render_frame(&images);
}
```

`platter` is small and can be plugged into your own game framework or game project if you want to harness `async/await` to load files on desktop and web. It will be a major part of the new Quicksilver application lifecyle!

That's Day 2 down, but there are 6 more days of Quicksilver Chanukah:

1. [Yesterday's introduction post](../quicksilver-chanukah-2019)
2. This post!
3. [`gestalt`: An API to manage bundling and saving data locally on desktop and web](../quicksilver-chanukah-2019-day-3)
4. [`blinds`: An easy-to-use async wrapper of `winit`](../quicksilver-chanukah-2019-day-4)
5. [`golem`: An opinionated mostly-safe graphics library for desktop and web GL](../quicksilver-chanukah-2019-day-5)
6. [Changes to the Quicksilver application lifecycle](../quicksilver-chanukah-day-6)
7. Changes to the Quicskilver graphics API
8. An overview of the work on web support this yea5
