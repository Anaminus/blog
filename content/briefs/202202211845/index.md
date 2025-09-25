+++
title = "Sun rays Mk II"
date = 2022-02-21 18:45:36
tags = ["roblox", "gamedev"]
+++

Sun rays with much better performance.

![](00.mp4)

Lot to unpack here. Each ray is a triangle within a MeshPart. Each MeshPart
groups together 85 triangles. Each vertex has a corresponding Bone, so the
triangles can be positioned independently and arbitrarily. Rays are produced by
allocating tris from the 85-tri meshes. A new MeshPart is created once all the
tris from the previous MeshPart are used.

The ForceField material is utilized here. As previously mentioned, the
orientation of vertex normals affects the appearance of the force field. Each
vertex corresponds to a bone. For each tri, one of the bones is designated as a
"roller".

The roller bone is moved out of the MeshPart into a separate Part that is
attached with a BallSocketConstraint at the original location. This part has
another attachment with a randomized orientation. This randomized attachment is
assigned to an AlignOrientation.

The other end of the AlignOrientation is assigned to a single attachment. The
brick in the video contains this attachment. The rotation of this brick is
directly controlling the "movement" of the rays.

The extra attachment with the randomized orientation is necessary so that the
appearance of force fields are spread out and unaligned. Assemblies with
BallSockets are necessary so that the bones are detected correctly while
allowing free orientation.

Now that I think about, the whole thing is driven entirely by physics. The only
script running continuously is to rotate the brick, but that's only because the
AngularVelocity constraint was being finicky.

Driving the control brick through physics doesn't work well, because the rollers
seem to fall asleep or otherwise get stuck. Updating it via script seems to keep
them awake.
