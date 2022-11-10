+++
title = "Considering options"
description = "or: How to implement an Optional type in Go"
date = 2022-11-10 06:00:00
+++

*or: How to implement an Optional type in Go*

I'm in the middle of rewriting [rbxfile][rbxfile]. A rewrite gives the
opportunity to right any wrongs.

One of the wrongs was my approach to Optional types. That is not to say that it
was incorrect, but it perhaps wasn't considered as carefully as it could have
been. Moreover, Go didn't have generics at the time, so there were fewer good
options available.

I currently have the Optional type implemented as a struct with a Type and a
Value field:

```go
type Optional struct {
	typ   Type
	value Value
}
```

A Type is an enum representing a limited set of data types. A Value is an
interface containing a value of one of these types (a part of the contract is a
`Type()` method that returns the Type).

The some-ness of the Optional is indicated by the Value being non-nil. When it
is nil, the additional Type field is included so that a none-y Optional still
has a type. These fields are encapsulated to prevent things from getting weird.

Overall, pretty clunky. This is what we must do when generics aren't a thing.

Once generics landed, more possibilities became available. I tried grappling
with them a few months ago, but wasn't able to settle on anything. Not only was
I still figuring out generics, but there were always questions of "What if this
implementation prevents me from using it in some necessary way? What if that
other implementation does the same thing, but for different requirements?" The
real problem was that the requirements were poorly understood.

Now that I'm rewriting the whole thing, I'm able to take a step back and look at
the full picture. With a better view, I was able to come up with the following
requirement:

- I need to be able to type-switch over the inner type of an optional.

That is, once a type has been determined to be *some kind* of Optional, I then
need to be able to determine that inner type. For example:

```go
func InspectValue(value Value) {
	switch value := value.(type) {
	case String:
		fmt.Println("it's a string!")
	case CFrame:
		fmt.Println("it's a CFrame!")
	case Optional:
		fmt.Println("it's optional!")
		v, _ := value.Optional()
		if _, ok := v.(Optional); !ok {
			InspectValue(v)
		}
	}
}
```

This would recursively cover the following types:
- `String`
- `CFrame`
- `Optional<String>`
- `Optional<CFrame>`

The keyword being "recursive". It's a lot easier if I don't have to reimplement
cases for each type of Optional that might crop up.

Once you have at least one requirement, it becomes infinitely easier to think
about the implementation. Whether it is good or bad can actually be answered.

Additionally, the various possible implementations for Optional have had time to
gestate in my mind. I was able to determine that there are two separated
concerns: an interface for the optional, and the implementations of this
interface.

## Interfaces
On the interface side, there are two possibilities:
- Typed interface
- Untyped interface

### Typed interface
The typed interface has a method where the optional returns its inner type
directly. It can be defined like so:

```go
type TypedOptional[T any] interface {
	TypedOptional() (value T, ok bool)
}
```

That is, for an Optional with inner type T, the method returns a value of type
T.

How does this fit into the requirements? Well, not very well. It isn't possible
to have *any kind* of optional as a case. The inner type of the optional must be
known beforehand. The inspector would have to look like this:

```go
func InspectValue(value Value) {
	switch value := value.(type) {
	case String:
		fmt.Println("it's a string!")
	case CFrame:
		fmt.Println("it's a CFrame!")
	case Optional[String]:
		fmt.Println("it's a string?")
		v, _ := value.Optional()
		InspectValue(v)
	case Optional[CFrame]:
		fmt.Println("it's a CFrame?")
		v, _ := value.Optional()
		InspectValue(v)
	}
}
```

Now, if I did need this kind of switching, then this interface would be great
to have. For now, though, I'll keep it off to the side.

### Untyped interface
The untyped interface has a similar method, except that the value is returned
through an empty interface:

```go
type UntypedOptional interface {
	UntypedOptional() (value any, ok bool)
}
```

Unlike the typed variation, this meets the requirements quite nicely. The
interface has no type parameters, and the method returns an interface that can
be type-switched on.

## Implementations
On the implementation side, there are also two possibilities:
- Unified
- Separated

Something to note is that both implementations can implement either kind of
interface.

### Unified type
The unified implementation consists of one type that embeds the state of the
Optional.

```go
type UnifiedOptional[T any] struct {
	Value T
	Ok    bool
}
```

Implementing each interface is straightforward:

```go
func (o UnifiedOptional[T]) UntypedOptional() (value any, ok bool) {
	return o.Value, o.Ok
}

func (o UnifiedOptional[T]) TypedOptional() (value T, ok bool) {
	return o.Value, o.Ok
}
```

Then I can create some constructors for each kind of optional. Two for Some and
None directly, and then another for specifying indirectly via boolean:

```go
func UnifiedSome[T any](value T) UnifiedOptional[T] {
	return UnifiedOptional[T]{Value: value, Ok: true}
}

func UnifiedNone[T any]() UnifiedOptional[T] {
	return UnifiedOptional[T]{Ok: false}
}

func UnifiedNewOptional[T any](value T, ok bool) UnifiedOptional[T] {
	return UnifiedOptional[T]{Value: value, ok: ok}
}

var (
	UnifiedDirectSome   = UnifiedSome(42)
	UnifiedDirectNone   = UnifiedNone[int]()
	UnifiedIndirectSome = UnifiedNewOptional(42, true)
	UnifiedIndirectNone = UnifiedNewOptional(0, false)
)
```

### Separated types
The separated implementation consists of separate types for Some and None:

```go
type SeparatedNone[T any] struct{}

type SeparatedSome[T any] struct{ Value T }
```

Each type implements each interface:

```go
func (o SeparatedNone[T]) UntypedOptional() (value any, ok bool) {
	var v T
	return v, false
}

func (o SeparatedNone[T]) TypedOptional() (value T, ok bool) {
	var v T
	return v, false
}

func (o SeparatedSome[T]) UntypedOptional() (value any, ok bool) {
	return o.Value, true
}

func (o SeparatedSome[T]) TypedOptional() (value T, ok bool) {
	return o.Value, true
}
```

Here, I only need one constructor, for specifying indirectly. Direct optionals
can be created via their respective composite literal:

```go
func SeparatedNewOptional[T any](value T, ok bool) TypedOptional[T] {
	if ok {
		return SeparatedSome[T]{Value: value}
	} else {
		return SeparatedNone[T]{}
	}
}

var (
	SeparatedDirectSome   = SeparatedSome{Value: 42}
	SeparatedDirectNone   = SeparatedNone[int]{}
	SeparatedIndirectSome = SeparatedNewOptional(42, true)
	SeparatedIndirectNone = SeparatedNewOptional(0, false)
)
```

## Pick something
The separated implementation seems like the most elegant. It's nice that the
state of the optional is stored in the type instead of as data.

The drawback is that it wouldn't work well with only the untyped interface. If I
wanted the option to assert specific types of optionals, like `Optional[String]`
or `Optional[CFrame]`, I wouldn't be able to, because those types don't exist. I
would have to assume the state of the optional as well as the type, such as
`Some[String]` or `None[CFrame]`.

However, if I elect to have both the typed and untyped interfaces at the same
time, then this option remains open. Each interface requires a separate method,
and each type implements both methods. To assert, I just have to use the typed
interface instead of an implementation, such as `TypedOptional[CFrame]`.
Interestingly, this still leaves the option of asserting specific states, too.

Can you get anymore flexible? Yes, actually. There's still the cases of
optionals with a specific state, but any type. This can be done by extending the
untyped interface with two more:

```go
type SomeOptional interface{
	UntypedOptional
	Some()
}
type NoneOptional interface{
	UntypedOptional
	None()
}

func (o SeparatedNone[T]) None() {}

func (o SeparatedSome[T]) Some() {}
```

With these final additions, cases for all possible combinations can be made:

```go
switch value.(type) {
case T:                // T
case any:              // ?
case None[T]:          // None<T>
case NoneOptional:     // None<?>
case Some[T]:          // Some<T>
case SomeOptional:     // Some<?>
case TypedOptional[T]: // Optional<T>
case UntypedOptional:  // Optional<?>
}
```

Now the problem is simply deciding on actual names for all of thes-

Maybe I'll reconsider...

[rbxfile]: https://github.com/robloxapi/rbxfile
