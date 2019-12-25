---
title: "A Quicksilver Chanukah, Day 3: Gestalt"
date: 2019-12-24
toc: false
images:
tags:
  - quicksilver
  - quicksilver-chanukah-2019
  - quicksilver-2019
  - gestalt
---

Often your application might want to store gamestate or configurations, but the web has no filesystem access. Additionally, you don't want to just dump files in the user's home directory; each desktop operating system has a different preferred location for storing application-specific data, and often different locations for different kinds of data.

Introducing [`gestalt`](https://crates.io/crates/gestalt), a library that bundles up your data and stores it away so you don't have to. A sample to save some game state:

```rust
use gestalt::{Location, save, load};
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize)]
struct Player {
    name: String,
    score: u32
}

let player1 = Player { name: "Bob".to_string(), score: 21 };
save(Location::Cache, "mygame", "player1", &player1).expect("Could not save Player 1");

let player2 = Player { name: "Alice".to_string(), score: 200 };
save(Location::Cache, "mygame", "player2", &player2).expect("Could not save Player 2");

// Now reload.
let player1 = load::<Player>(Location::Cache, "mygame", "player1").expect("Could not load Player 1");
let player2 = load::<Player>(Location::Cache, "mygame", "player2").expect("Could not load Player 2");
```

Getsalt creates a Key/Value store, backed by the filesystem on desktop and the web storage API. The key is formed by `Location`, which determines the kind of data (`Location::Cache`, `Location::Data`, and `Location::Config`), the application name, and a string which identifies the name of the data chunk. The value is anything that implements `Serialize` from [`serde`](https://crates.io/crates/serde).

`Cache` is for data that can be cleared between runs of your application; `Data` is for blobs like save states, and `Config` is for application settings. `gestalt` uses [`dirs`](https://crates.io/crates/dirs) to ensure that it saves your data in the right place for each, regardless of your user's operating system.

That's Day 3 done: `gestalt` is pretty small! Tomorrow's library, `blinds`, starts to get quite a bit bigger.

1. [Yesterday's introduction post](../quicksilver-chanukah-2019)
2. [`platter`: An async file-loading API for desktop and web](../quicksilver-chanukah-2019-day-2)
3. This post!
4. `blinds`: An easy-to-use async wrapper of `winit`
5. `golem`: An opinionated mostly-safe graphics library for desktop and web GL
6. Changes to the Quicksilver application lifecycle
7. Changes to the Quicskilver graphics API
8. An overview of the work on web support this yea5
