+++
title = "On CFrame vectors"
description = "Correcting documentation on CFrames."
date = 2023-01-29 12:00:00
tags = ["roblox"]
+++

In Roblox, the [CFrame][CFrame] type has several "Vector" fields representing
the directions of various axes of the CFrame. There are two sets of 3 vectors:

- [RightVector][RightVector], [UpVector][UpVector], [LookVector][LookVector]
- [XVector][XVector], [YVector][YVector], [ZVector][ZVector]

Roblox's documentation claims that these fields represent the rows and columns
of the CFrame's rotation matrix. The X, Y, Z fields are the rows, and the Right,
Up, Look fields are the columns. **This is very significantly incorrect**.
Inspecting the values of each component reveals it so:

	CFrame.identity:GetComponents()
		0 0 0 1 0 0 0 1 0 0 0 1

	CFrame.identity.Position
		0 0 0

	CFrame.identity.XVector
		1 0 0

	CFrame.identity.YVector
		0 1 0

	CFrame.identity.ZVector
		0 0 1

	CFrame.identity.RightVector
		1 0 0

	CFrame.identity.UpVector
		0 1 0

	CFrame.identity.LookVector
		-0 -0 -1

What we can see is that the X-, Y- and ZVector fields correspond directly to the
components:

	Fields      [Position][XVector][YVector][ZVector]
	Components  0, 0, 0,  1, 0, 0, 0, 1, 0, 0, 0, 1

We also notice some other things:
- RightVector and UpVector appear to be redundant with XVector and YVector.
- LookVector always equal to -ZVector.

Why would it be like this? Is the implementation incorrect?

Nope. The explanation is that the X-, Y-, and ZVector fields represent the raw
components of the CFrame matrix, while the Right-, Up- and LookVector fields
represent more practical values.

Originally, CFrames had only the LookVector field (stylized as `lookVector`). It
represents the most interesting vector, being the "front" face of the CFrame, or
the direction the CFrame was "looking". Very useful for getting things to look
at or move towards other things. For whatever reason, Roblox defined the front
face to be the *complement* of the Z direction.

Eventually, the other vector fields were added. RightVector and UpVector were
added as counterparts to LookVector. However, to correctly derive the raw
components from these vectors, one would have to remember to invert the
LookVector.

```lua
local right = cframe.RightVector
local up = cframe.UpVector
local lookaway = -cframe.LookVector

local r00, r01, r02 = right.X, right.Y, right.Z
local r10, r11, r12 = up.X, up.Y, up.Z
local r20, r21, r22 = lookaway.X, lookaway.Y, lookaway.Z
```

Because this would be confusing and easy to forget, the ZVector field was added
to represent the raw Z direction, along with XVector and YVector as
counterparts. This explains the redundancy of XVector/RightVector and
YVector/UpVector.

```lua
local x = cframe.XVector
local y = cframe.YVector
local z = cframe.ZVector

local r00, r01, r02 = x.X, x.Y, x.Z
local r10, r11, r12 = y.X, y.Y, y.Z
local r20, r21, r22 = z.X, z.Y, z.Z
```

To summarize, RightVector is always equal to XVector, UpVector is always equal
to YVector, and LookVector is always equal to the complement of ZVector. And
don't let the documentation let you think otherwise.

	RightVector ==  XVector
	UpVector    ==  YVector
	LookVector  == -ZVector

[CFrame]: https://create.roblox.com/docs/reference/engine/datatypes/CFrame
[RightVector]: https://create.roblox.com/docs/reference/engine/datatypes/CFrame#RightVector
[UpVector]: https://create.roblox.com/docs/reference/engine/datatypes/CFrame#UpVector
[LookVector]: https://create.roblox.com/docs/reference/engine/datatypes/CFrame#LookVector
[XVector]: https://create.roblox.com/docs/reference/engine/datatypes/CFrame#XVector
[YVector]: https://create.roblox.com/docs/reference/engine/datatypes/CFrame#YVector
[ZVector]: https://create.roblox.com/docs/reference/engine/datatypes/CFrame#ZVector