+++
title = "Collision detection"
date = 2021-12-13 00:43:31
tags = ["roblox", "gamedev"]
+++

Continuous collision detection. Very happy to finally have this working. No
raycasts. All Luau and a bunch of maths.

![](00.mp4)

The position of the yellow ball is the white ball plus its velocity. The green
ball is the yellow ball plus the new velocity.

It only does spheres vs triangles since that’s all I need, but it’s based off of
[GeometricTools][gt], which has a number of different shapes.

[gt]: https://github.com/davideberly/GeometricTools
