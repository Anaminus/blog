+++
title = "Sun rays"
date = 2022-02-20 15:47:20
tags = ["roblox", "gamedev"]
+++

Sun rays.

![](00.mp4)

Each ray is just a stretched cube mesh with an almost transparent ForceField
material. The mesh is just for surface smoothing, so a simple Union could be
used as well, though it's less convenient to resize. Each part is rotated
continuously.

Rays are cast from a defined plane using GetSunDirection, so some occlusion can
be achieved.

![](02.mp4)

In practice, it seems like using meshes and ForceFields is excessive. Regular
transparent parts seem to produce more or less the same effect.

Retrying with a simple triangle mesh has much better performance. An interesting
side-effect of using a mesh with bones is that the shape and appearance of the
ForceField can be controlled by the vertex normals.

Rotating the vertex normals instead of the part produces a subtle
rolling-fog-like effect. Not sure if it will show up in the video, but it's
really cool. Unfortunately, the performance is awful. Bone modification doesn't
seem to be as optimized as it could be.

![](01.mp4)

According to the profiler, modifying a Bone's CFrame once involves 27 instances
of "Loader", each involving a call to "IsA". Definitely doesn't seem right.

Apparently the ForceField material can be affected by vertex color alphas, but
Blender doesn't support them???

A further optimization is to have multiple triangles per MeshPart. A mesh can
have up to 256 bones, which amounts to 85 triangles. This reduces the above
scene of ~400 parts down to just 6.
