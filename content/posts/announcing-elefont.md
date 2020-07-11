+++
title = "A New Quicksilver Crate, Elefont"
date = 2020-07-10
[taxonomies]
tags = ["quicksilver", "elefont"]
+++

This blog post has been a long time delayed, but late is better than never! The Quicksilver alpha is humming along, with bugs and API problems being addressed. While that work continues, I want to unveil `elefont`, a crate that I've developed for font caching.

<!-- more -->

## What is Elefont?

Elefont is a glyph caching layer, intended to reduce round-trips to the GPU when rendering text.
When a glyph is sent to the GPU, it is stored on a texture and re-used until the cache needs to be refreshed.
Elefont is also designed to support many different font providers, and many different graphics backends: you can currently choose to render with [rusttype](https://gitlab.redox-os.org/redox-os/rusttype) or [fontdue](https://github.com/mooman219/fontdue/), and there are plans to [support bitmap fonts](https://github.com/ryanisaacg/elefont/issues/5) and [operating-system-specific font toolkits via font_kit](https://github.com/ryanisaacg/elefont/issues/3). 
It is designed for use primarily in games, because it was written for Quicksilver, and the design reflects that. A GUI toolkit can probably choose a specific font rendering toolkit and use it throughout, but games tend to have more idiosyncratic text requirements (like bitmap fonts.)

## What is Glyph Caching?

Glyph caching is an approach to font rendering that stores each "glyph" (font image) on the GPU. 

<aside>

Human writing is vast and complex, and so are the systems we have built to digitize it. A "glyph" is not the same as a "character;" this may surprise people who exclusively use Latin script. For example, Unicode emoji support [skin-tone modifiers](https://emojipedia.org/emoji-modifier-sequence/). These are separate Unicode code-points that are placed *after* the emoji code-point, and modify the image that is produced. Other familiar examples include all-caps fonts: 'A' and 'a' are different characters, but may be represented by the same glyph. For more on the complexity of text rendering, see [Text Rendering Hates You](https://gankra.github.io/blah/text-hates-you/).

</aside>

Most fonts are stored in files like 'TTF' or 'OTF'. These are called 'vector fonts,' because the glyphs are stored as a series of strokes; similar technology powers SVG images or Adobe Flash graphics. Every time you render a glyph from one of these fonts, the font system uses a 'rasterizer' to turn the strokes of the glyph into a grid of pixel color values. This is a fairly expensive operation, so glyph caches store this resulting bitmap instead of re-generating it every time the character is used. Elefont helps in this case by keeping track of a map of glyphs to their rendered counterparts.

Even if you're using a "bitmap font" (a font defined by pixel image data instead of vector graphics), you still have to send that image data to the GPU. With the machinery already in place to map from glyphs to rendered images, Elefont can help in this case by keeping track of a map of glyphs to their GPU-side counterparts. This is a high priority for a future version, but isn't written yet.

## Why not use X?

Initially, I didn't expect to write Elefont or anything similar. I knew that Quicksilver 0.4 would need glyph caching, because fonts and text were a significant pain point in Quicksilver 0.3. What I did not find was anything that fit my requirements:

1. Work with any graphics backend. Quicksilver uses `golem`, my abstraction layer over OpenGL, so being tied to another graphics library was a no-go
2. Work with any font provider. Users may have bitmap fonts or vector fonts; they may want system-native font rendering (with some web fallback); they may want the more mature rusttype or the more ambitious fontdue, etc.

There were other options that pass requirement 1, chief among them [glyph_brush](https://crates.io/crates/glyph-brush). However, glyph_brush is tied into rusttype, and doesn't seem to be interested in supporting bitmap fonts or other rasterizers. For many uses, glyph_brush is the right choice! However, it just couldn't fit my needs. Bitmap fonts are a fairly common request when rendering for games, and a design that precludes them is a missed opportunity.

## What Comes Next?

(Hopefully soon) I'll be putting out a State of Quicksilver update, that generally addresses progress the project has made this year!
