---
title: "A Quicksilver Chanukah, Day 5: Golem"
date: 2019-12-26
toc: false
images:
tags:
  - quicksilver
  - quicksilver-chanukah-2019
  - quicksilver-2019
  - golem
---

## What is `golem`?

`golem` is a mostly-safe graphics API that targets OpenGL 3.2 and WebGL 1 that helps make writing GL less painful

## Why `golem` and not X

If you're just interested in what `golem` is and how it works, feel free to skip this section!

The previous three days I've introduced new crates that mostly stood on their own. I'm not aware of any fleshed-out alternatives, so creating those libraries myself doesn't require much explanation. `golem`, however, is a different story: there aren't exactly a shortage of Rust graphics libraries. There are a variety of reasons as to why none of them fit Quicksilver's needs, most of which revolve around support for older hardware or older APIs. For brevity, I'd recommend you check out [icefox's guide to the existing graphics options](https://wiki.alopex.li/AGuideToRustGraphicsLibraries2019).

I have a test device that I think is a good benchmark for something Quicksilver-powered games should be able to target, and it doesn't support Vulkan. This eliminates `ash` and `vulkano` straight off the bat, and `gfx-hal` as well. `ash` and `vulkano` target Vulkan directly, and `gfx-hal`'s OpenGL backend is not currently reliable enough (though I hope to see that improve!)

I also want to be able to support WebGL 1, but WebGL 1 is not a very modern version of OpenGL. This makes it, for good reaon, unattractive to other libraries (like `luminance`, which is targeting WebGL 2 in its upcoming web support). However, WebGL 2's [market share](https://caniuse.com/#feat=webgl2) still hovers at around 75%, which isn't ideal; notably missing from WebGL 2 support are the current verisons of Edge, Safari, and iOS Safari. Quicksilver recently transitioned to using WebGL 1 for better platform support, and I didn't want to regress by requiring a high version. This rules out `luminance`, and presumably `glium` (which has no current development towards web support as far as I'm aware.)

With just those two constraints I'm out of options, so I decided to write `golem`.

## Mostly-Safe?

Where possible, `golem` handles for you the complex state machine of OpenGL. In other places, it prevents you from using resources that haven't been properly set-up. In these cases, `golem` is safe with low-overhead, removing footguns from OpenGL without slowing your code down. In others, more overhead would be required to keep the API free of `unsafe`. Currently only two functions are marked as `unsafe` in `golem`'s public API, but they are the only functions that draw things to the screen! This means using `unsafe` code as a consumer of `golem` is unavoidable, hence "mostly-safe."

## A Code Snippet

Hopefull if you're familiar with graphics APIs, this code snippet to draw a triangle to the screen will look a little familiar. If not, the comments should help you follow along!

```rust
use blinds::*;
use golem::{
    Attribute, AttributeType, Context,
    Dimension::{D2, D4},
    ElementBuffer, GeometryMode, GolemError, ShaderDescription, ShaderProgram, VertexBuffer,
};

// The application loop, powered by the blinds crate
async fn app(
    window: Window,
    ctx: glow::Context,
    mut events: EventStream,
) -> Result<(), GolemError> {
    // Create a context from 'glow', GL On Whatever
    let ctx = &Context::from_glow(ctx)?;

    #[rustfmt::skip]
    // This is the data that represents the triangle
    // It's arranged how it will be passed to the GPU: each position represented as two f32 values,
    // followed by each color represented as 4 f32 values. The positions are on a scale from -1.0
    // to 1.0, which represents the viewport in OpenGL. The colors are represented as R, G, B, A,
    // on a scale from 0.0 to 1.0
    let vertices = [
        // Position         Color
        -0.5, -0.5,         1.0, 0.0, 0.0, 1.0,
        0.5, -0.5,          0.0, 1.0, 0.0, 1.0,
        0.0, 0.5,           0.0, 0.0, 1.0, 1.0
    ];
    // This is the data that indicates how to draw the vertices
    // For a simple example of one triangle, we don't gain much from this. Any order of these three
    // points will give us the same triangle. However, if we add more points (to draw a square, for
    // example), then we can write each point once while using it in multiple triangles.
    let indices = [0, 1, 2];

    // Here we create the ShaderProgram, which is some code that runs on the GPU. It determines how
    // to turn our vertex data into an actual vertex that GL understands, and how to color each
    // 'fragment' (essentially a pixel). These are each their own little program, where the
    // information from the vertex shader is fed into the fragment shader.
    // For the purposes of making sure the shaders match, and for ensuring compatibility on desktop
    // and web, the inputs are represented as data structures and then converted to shader
    // declarations at runtime.
    // The input to the shader program is fed to the vertex_input, so your vertex data's format
    // needs to match what you define in vertex_input
    let mut shader = ShaderProgram::new(
        ctx,
        ShaderDescription {
            // Take in to the shader a position (as a vector with 2 components) and a color (as a
            // vector with 4 components). This is the same format as 'vertices' above
            vertex_input: &[
                Attribute::new("vert_position", AttributeType::Vector(D2)),
                Attribute::new("vert_color", AttributeType::Vector(D4)),
            ],
            // Pass to the fragment shader the color
            // OpenGL will actually smoothly interpolate between different vertex values for us, so
            // a red vertex and a blue vertex will have a gradient between them
            fragment_input: &[Attribute::new("frag_color", AttributeType::Vector(D4))],
            // Uniforms represent a value that's the same for the entire shader; we don't need any
            // here. If you're rendering images or applying transformations to your entire draw
            // call, use uniforms!
            uniforms: &[],
            // A program written in GLSL that uses the inputs and outputs defined above
            // There's also a hard-coded output called gl_Position
            vertex_shader: r#" void main() {
            gl_Position = vec4(vert_position, 0, 1);
            frag_color = vert_color;
        }"#,
            // The fragment shader has a hard-coded output: gl_FragColor
            fragment_shader: r#" void main() {
            gl_FragColor = frag_color;
        }"#,
        },
    )?;

    // Create buffer objects, which we use to transfer data from the CPU to the GPU
    let mut vb = VertexBuffer::new(ctx)?;
    let mut eb = ElementBuffer::new(ctx)?;
    // Set the data of the buffer to be our vertices and indices from earlier
    vb.set_data(&vertices);
    eb.set_data(&indices);
    // Prepare the shader for operations: shaders will raise errors if you forget to bind them
    shader.bind();
    // Clear the screen
    ctx.clear();
    unsafe {
        // Using our buffers, draw our triangle
        // We could also interpert our indices as Lines or a variety of other shape options:
        // nothing binds us to necessarily using Triangles, even though they're the most common
        // shape in graphics
        shader.draw(&vb, &eb, 0..indices.len(), GeometryMode::Triangles)?;
    }
    // Show our data to the window
    window.present();
    // Keep the window open and responsive until the user exits
    loop {
        events.next_event().await;
    }
}

// Run our application!
fn main() {
    run_gl(Settings::default(), |window, gfx, events| {
        async move { app(window, gfx, events).await.unwrap() }
    });
}
```

(This is also an example of [yesterday's `blinds`](../quicksilver-chanukah-2019-day-4) and its OpenGL support.)

## Further Development

`golem` is not intended to be a fully-featured wrapper around all of OpenGL. It's intended to be enough to write fairly complex 2D graphics, or fairly simple 3D graphics. That said, there is a lot that's missing! Some of it is documented on the [issue tracker](https://github.com/ryanisaacg/golem/issues), but I'm sure that I've missed other things. Nearly everything I need for Quicksilver is done, but your use case might be different. Feel free to open an issue discussing a feature addition, or try and experiment with a fork for larger changes.

`golem` is the last new crate to announce, which means tomorrow is about what these crates enable: the new version of Quicksilver.

1. [The introduction post](../quicksilver-chanukah-2019)
2. [`platter`: An async file-loading API for desktop and web](../quicksilver-chanukah-2019-day-2)
3. [`gestalt`: An API to manage bundling and saving data locally on desktop and web](../quicksilver-chanukah-2019-day-3)
4. [`blinds`: An easy-to-use async wrapper of `winit`](../quicksilver-chanukah-2019-day-4)
5. This post!
6. [Changes to the Quicksilver application lifecycle](../quicksilver-chanukah-day-6)
7. Changes to the Quicskilver graphics API
8. An overview of the work on web support this year
