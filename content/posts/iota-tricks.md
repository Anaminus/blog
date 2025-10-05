+++
title = "Iota tricks"
description = "Using Go's iota for questionable reasons."
date = 2025-09-26 12:00:00
tags = ["go"]
+++

Go has a neat syntax called [iota][iota], which sequentially increments for
constants in the same group:

```go
const (
	A = iota // int = 0, defaults to int type
	B = iota // int = 1
	C        // int = 2, uses previous value, which is iota
	D        // int = 3
)
```

The most common pattern is to have the first define the rest:

```go
const (
	A uint = iota // uint = 0
	B             // uint = 1
	C             // uint = 2
	D             // uint = 3
	E             // uint = 4
)
```

For completely irrational reasons, I like to define a blank constant that uses
iota, then have the actual constants filled in automatically. The blank, which
is discarded, defines the expression to be used for the remaining values. Math
is allowed, so 1 can be subtracted to get the actual constants to have the
correct values:

```go
const (
	_ = iota - 1 // int = -1
	A            // int =  0
	B            // int =  1
	C            // int =  2
	D            // int =  3
)
```

Nice and consistent. There's a problem, though. An error occurs for unsigned
types, because they're not allowed to underflow:

```go
// cannot use iota - 1 (untyped int constant -1)
// as uint value in constant declaration (overflows)
const _ uint = iota - 1
```

This can be fixed by using the [max][max] function to clamp the value. Since
it's built-in, it can be used on constants:

```go
const (
	_ uint = max(0, iota-1) // uint = 0
	A                       // uint = 0
	B                       // uint = 1
	C                       // uint = 2
	D                       // uint = 3
)
```

## Flags
Flags are another common pattern, which involves defining powers of two:

```go
const (
	A = 1 << iota // int = 1
	B             // int = 2
	C             // int = 4
	D             // int = 8
)
```

When formatting this to use a blank constant, a similar problem arises:

```go
// invalid operation: negative shift count (iota - 1)
// (untyped int constant -1)
const _ = 1 << (iota - 1)
```

To fix this, instead of subtracting 1, divide by two, which has the same effect:

```go
const (
	_ = (1 << iota) / 2 // uint = 0
	A                   // uint = 1
	B                   // uint = 2
	C                   // uint = 4
	D                   // uint = 8
)
```

## Conclusion
In short,

```go
// For sequences.
const (
	_ = max(0, iota-1)
	A // 0
	B // 1
	C // 2
	D // 3
)
// For flags.
const (
	_ = (1 << iota) / 2
	A // 1
	B // 2
	C // 4
	D // 8
)
```

[iota]: https://go.dev/ref/spec#Iota
[max]: https://go.dev/ref/spec#Min_and_max
