---
title: "Rust Quake Rasterizer"
date: 2022-07-07T22:58:33-03:00
draft: false
thumbnail: screenshot1.png
---

Decided to implement a software rasterizer in Rust, and what better way to do it than using Quake 1 levels?

Still working on this project but so far it supports:

* Vertex transform pipeline
* Rasterizer using barycentric coordinates (as opposed to Quakes scanline rasterizer)
* PVS for determining the leafs to render
* Texture mapping (using just mip 0 for now)

I still have lots of work to do on it, I haven't even started looking into optimizing it much. I did try
rasterizing triangles using tiles to skip big portions of "inside" pixels, but I didn't get as much improvement 
as I meant, so for now its disabled.

![Image alt Main](screenshot1.png "Software Rasterizer")
![Image alt Main](screenshot2.png "Software Rasterizer")

Repository available at: https://github.com/xamado/rusted-quake

