+++
title = "1-bit LÖVE"
date = 2022-12-30 02:34:12
tags = ["gamedev"]
+++

Having some fun with LÖVE.

![](00.mp4)

Here's a view of the chunk buffer demonstrating simplified chunk loading:

![](01.mp4)

The white area is the viewport. Chunks are updated only when the focus leaves
the yellow area. Each corner of the blue area determines which chunks are
loaded.
