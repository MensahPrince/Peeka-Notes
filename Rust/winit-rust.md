---
id: 6v7lwwxlhd
title: "Winit-Rust"
cover: "_assets/344db9c32c2fceb2d90ec562094c31db.jpg"
cover_position: 50
sort: 1000
version: 1
winit: Desktop Window in Rust
---

<callout>

winit = "0.30.13"

</callout>

---

Winit is a cross-platform window creation and event loop management library.

## Building windows

Before you can create a [`Window`](https://docs.rs/winit/latest/winit/window/struct.Window.html), you first need to build an [`EventLoop`](https://docs.rs/winit/latest/winit/event_loop/struct.EventLoop.html). This is done with the [`EventLoop::new()`](https://docs.rs/winit/latest/winit/event_loop/struct.EventLoop.html#method.new) function.

```
use winit::event_loop::EventLoop;

fn main() {
    let event_loop = EventLoop::new().unwrap();
    // ...
}
```

Then you create a [`Window`](https://docs.rs/winit/latest/winit/window/struct.Window.html) with [`create_window`](https://docs.rs/winit/latest/winit/event_loop/struct.ActiveEventLoop.html#method.create_window).

```
```
