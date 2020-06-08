+++
title = "Website Makeover, Again"
date = 2020-05-24
[taxonomies]
tags = ["site", "meta"]
+++

Out with the old, in with the new! After some frustration with Hugo, I've moved to [Zola](https://www.getzola.org/) and some light CSS.

<!-- more -->

I wanted to write a new blog post regarding [elefont](https://github.com/ryanisaacg/elefont) and Quicksilver's new text rendering system, but I  was dreading making a post on the site. Hugo had proved opaque and fragile for me, and its blizzard of configuration options wasn't making things much easier. Plus, something small broke when I tried to use the latest Hugo version on Windows to re-build the site. I wasn't much interested in tracking down a changelog and bringing my blog up to the latest version, so I decided Hugo had to go.

Initially I considered writing my own static site generator. _How hard could it be?_ I asked myself. Then I thought about generating an RSS feed, and parsing front-matter, and maybe setting up a SASS compilation pipeline. _No thanks!_ I instead settled on Zola, primarily because it seems to be the right combination of simple and expressive. Secondarily, it's written in Rust, which I appreciate. Using Zola has been very nice so far; I feel more like I'm programming a little language designed to make website-making easy. Previous experiences with Hugo and Jekyll felt more like wrestling the generator into doing what I wanted; Zola is a breath of fresh air by comparison.

Currently the site is very light on styling. So far I like the look of it, but for how long is anyone's guess.

I hope this means I can get some posts up about Quicksilver soon; there's been a lot of evolution since the end of last year, when I was just starting on the new alpha versions. I ended up forgoing a State of Quicksilver 2020, for a few reasons:

- [The Quicksilver Chanukah posts](../quicksilver-chanukah-2019) covered a lot of my plans
- I wanted to wait until the 0.4 release is closer to feature-complete
- And mostly, I've been very busy

Post-script update, as of June 8th 2020: The blog is a bit more automatic! Previously I was just pushing up new built versions of the site to Github Pages, but that gets tedious. I set up Github Actions to run on any push to ryanisaacg/blog on master; now the blog is automatically built and deployed.
