+++
title = "CFrame mnemonics"
date = 2023-01-31 12:00:00
+++

## Methods
Certain methods on CFrame have an equivalent operator-based expression, which
can be useful for understanding how they work:

| Method                     | Expression                            |
|----------------------------|---------------------------------------|
| `A:ToWorldSpace(B)`        | `A * (B::CFrame)`                     |
| `A:ToObjectSpace(B)`       | `A:Inverse() * (B::CFrame)`           |
| `A:PointToWorldSpace(B)`   | `A * (B::Vector3)`                    |
| `A:PointToObjectSpace(B)`  | `A:Inverse() * (B::Vector3)`          |
| `A:VectorToWorldSpace(B)`  | `A.Rotation * (B::Vector3)`           |
| `A:VectorToObjectSpace(B)` | `A:Inverse().Rotation * (B::Vector3)` |

## Space-conversion analogy
CFrame multiplication is not equivalent to addition. However, certain aspects
can be reused in order to remember how it works.

Pretend that `A * B` is analogous to `B + A`. Also pretend that `A:Inverse()` is
analogous to `-A`. There is no subtraction, but the formula `B - A` can be
rewritten as `-A + B`. So, when we see the expression `A:Inverse() * B`, it can
be thought of as a sort of `B - A`.

This explains ToObjectSpace. It "subtracts" the origin `A` from the subject `B`
to get the location of `B` relative to `A`. ToWorldSpace is the reverse; the
origin is added back to the subject to get the real world location.

| Method               | Analogy | Rewritten | Actual            |
|----------------------|--------:|----------:|-------------------|
| `A:ToWorldSpace(B)`  | `B + A` |  `A + B`  | `A * B`           |
| `A:ToObjectSpace(B)` | `B - A` | `-A + B`  | `A:Inverse() * B` |
