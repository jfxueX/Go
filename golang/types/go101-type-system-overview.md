[Overview Of Go Type System][1]
===============================

[1]: https://go101.org/article/type-system-overview.html
[2]: basic-types-and-value-literals.html 

This article will introduce all kinds of types in Go. All kinds of concepts in 
Go type system will also be introduced. Without knowing these concepts, it is 
hard to have a thorough understanding of Go.

### Concept: Basic Types

Built-in basic types in Go have been introduced in [built-in basic types and 
basic value literals][2]. For completeness of the current article, these built-
in basic types are re-listed here.

  - Built-in string type: `string`.
  - Built-in boolean type: `bool`.
  - Built-in numeric types:
      - `int8`, `uint8` (`byte`), `int16`, `uint16`, `int32` (`rune`),
        `uint32`, `int64`, `uint64`, `int`, `uint`, `uinptr`.
      - `float32`, `float64`.
      - `complex64`, `complex128`.

Except [string types](string.html), Go 101 article series will not try to 
explain more on other basic types.

### Concept: Composite Types

Go supports following composite types:

  - [pointer types][3] - C pointer alike.
  - [struct types][4] - C struct alike.
  - [function types][5] - functions are first-class types in Go.
  - [container types][6]:
      - array types - fixed-length container types.
      - slice type - dynamic-length and dynamic-capacity container types.
      - map types - maps are associative arrays (or dictionaries). The standard 
        Go compiler implements maps as hashtables.
  - [channel types][7] - channels are used to synchronize data among goroutines 
    (the green threads in Go).
  - [interface types][8] - interfaces play a key role in reflection and 
    polymorphism.

[3]: pointer.html
[4]: struct.html
[5]: function.html
[6]: container.html
[7]: channel.html
[8]: interface.html

Composite types may be denoted as their respective type literals.  Following are 
some literal representation examples of all kinds of composite types.

```go
// Assume T is an arbitrary type and Tkey is
// a type supporting comparison (== and !=).

*T         // a pointer type
[5]T       // an array type
[]T        // a slice type
map[Tkey]T // a map type

// a struct type
struct {
    name string
    age  int
}

// a function type
func(int) (bool, string)

// an interface type
interface {
    Method0(string) int
    Method1() (int, bool)
}

// some channel types
chan T
chan<- T
<-chan T
```

*([Comparable and uncomparable types](#types-not-support-comparison) will be 
explained below.)*

### Fact: Kinds Of Types

Each of the above mentioned basic and composite types corresponds to one kind of 
types. Besides these type kinds. The unsafe pointer types introduced in the 
[`unsafe` standard package][9] also belong to one kind of types in Go. So, up to 
now (Go 1.11), Go has 26 kinds of types.

[9]: https://golang.org/pkg/unsafe

### Syntax: Type Definitions

*(**Type definition**, or type definition declaration, was called **type 
declaration** and was the only type declaration way before Go 1.9. Since Go 1.9, 
type definition has become one of two forms of type declartions in Go. The other 
new form is called **type alias declaration**, which will be introduced in the 
next section.)*

In Go, we can define new types by using the following form. In the syntax, 
`type` is a keyword.

```go
// Define a solo new type.
type NewTypeName SourceType

// Define multiple new types together.
type (
    NewTypeName1 SourceType1
    NewTypeName2 SourceType2
)
```

New type names must be identifiers.

Note,

  - a new defined type and its respective source type in type definitions are 
    two distinct types.

  - two types defined in two type definitions are always two distinct types.

  - the new defined type and the source type will share the same underlying type 
    (see below for what are underlying types), and their values can be converted 
    to each other.

  - types can be defined within function bodies.

Some type definition examples:

```go
// The following new defined and source types are all basic types.
type (
    MyInt int
    Age   int
    Text  string
)

// The following new defined and source types are all composite types.
type IntPtr *int
type Book struct{author, title string; pages int}
type Convert func(in0 int, in1 bool)(out0 int, out1 string)
type StringArray [5]string
type StringSlice []string

func f() {
    // The names of the three defined types
    // can be only used within the function.
    type PersonAge map[string]int
    type MessageQueue chan string
    type Reader interface{Read([]byte) int}
}
```

### Syntax: Type Alias Declarations

*(**Type alias declaration** is one new kind of type declartions added since Go 
1.9.)*

As above mentioned, there are only two built-in type aliases in Go, `byte` 
(alias of `uint8`) and `rune` (alias of `int32`). They are the only two type 
aliases before Go 1.9.

Since Go 1.9, we can declare custom type alias by using the following syntaxes. 
The syntax of alias declaration is much like type definition, but please note 
there is a `=` in each type alias declaration.

```go
type (
    Name = string
    Age  = int
)

type table = map[string]int
type Table = map[Name]Age
```

Type alias names must be identifers. Like type definitions, type aliases can 
also be declared within function bodies.

By the above declarations, `Name` is an alias of `string`, they denote the same 
type. The same relation is for the other three pairs of type:

  - `Age` and `int`
  - `table` and `map[string]int`
  - `Table` and `map[Name]Age`

In fact, type `map[string]int` and `map[Name]Age` also denote the same type. So 
the four types involved in the last two type alias declarations all denote the 
same type.

Note, although `table` and `Table` denote the same type, `Table` is an exported 
type so it can be used by other packages but `table` can't.

### Concept: Defined Types vs. Non-Defined Types

A defined type is a type defined in a type definition or an alias of another 
defined type.

All basic types are defined. A non-defined type must be a composite type.

In the following example. alias type `C` and type literal `[]string` are both 
non-defined types, but type `A` and alias type `B` are both defined types.

```go
type A []string
type B = A
type C = []string
```

### Concept: Named Types vs. Unnamed Types

In Go,

  - if a type has a name, which must be an identifier, and the
    identifier is not the blank identifier `_`, then this type is called
    a named type. All basic types are named types. A declared type is
    also a named type only if its identifier is not the blank
    identifier.
  - if a type can't be represented by a pure identifier, then the type
    is an unnamed type. The composite types denoted by their respective
    type literals are all unnamed types.

An unnamed type must be a composite type, It is not true vice versa, for
a composite type may be a defined type or an alias type.

### Concept: Underlying Types

In Go, each type has an underlying type. Rules:

  - for built-in basic types, the underlying types are themselves.
  - the underlying type of `unsafe.Pointer` is itself.
  - the underlying types of an unnamed type, which must be a composite
    type, is itself.
  - in a type declaration, the new declared type and the source type
    have the same underlying type.

Examples:

```go
// The underlying types of the following ones are both int.
type (
    MyInt int
    Age   MyInt
)

// The following new types have different underlying types.
type (
    IntSlice = []int   // underlying type is []int
    MyIntSlice []MyInt // underlying type is []MyInt
    AgeSlice   []Age   // underlying type is []Age
)

// The underlying types of Ages and AgeSlice are both []Age.
type Ages AgeSlice
```

How to trace to the underlying type for a given user declared type? The
rule is, when a built-in basic type, `unsafe.Pointer` or an unnamed type
is met, the tracing will be stopped. Take the type declarations above as
examples, let's trace their underlying types.

    MyInt → int
    Age → MyInt → int
    IntSlice → []int
    MyIntSlice → []MyInt → []int
    AgeSlice → []Age → []MyInt → []int
    Ages → AgeSlice → []Age → []MyInt → []int

In Go,

  - types whose underlying types are `bool` are called **boolean
    types**;
  - types whose underlying types are any of built-in integer types are
    called **integer types**;
  - types whose underlying types are either `float32` or `float64` are
    called **floating-point types**;
  - types whose underlying types are either `complex64` or `complex128`
    are called **complex types**;
  - integer, floating-point and complex types are also called **numeric
    types**;
  - types whose underlying types are `string` are called **string
    types**.

The concept of underlying type plays an important role in [value conversions, 
assignments and comparisions in Go][10].

[10]: value-conversions-assignments-and-comparisons.html

### Concept: Values

An instance of a type is called a value, of the type. A type may have many 
values, one of them is the zero value of the type. Values of the same type share 
some common properties.

Each type has a zero value, which can be viewed as the default value of
the type. The predeclared `nil` identifier can used to represent as zero
values of slice, map, function, channel, pointer (including type-unsafe
pointer) and interface types. About more on `nil`, please read [nil in
Go](nil.html) later.

There are several kinds of value representation forms in code, including
[literals](basic-types-and-value-literals.html), [named
constants](constants-and-variables.html#constant),
[variables](constants-and-variables.html#variable) and
[expressions](expressions-and-statements.html), though the former three
can be viewed as special cases of the latter one.

Value may be [typed or
untyped](constants-and-variables.html#untyped-value).

All kinds of basic value literals have been introduced in in the article
[basic types and basic value
literals](basic-types-and-value-literals.html). There are two more kinds
of literals in Go, composite literals and function literals.

Function literals, as the name implies, are used to represent function
values. A [function
declaration](function-declarations-and-calls.html#declaration) is
composed of a function literal and a function identifier.

Composite literals are used to represent values of struct types and
container types (arrays, slices and maps), Please read [structs in
Go](struct.html) and [containers in Go](container.html) for details.

There are no literals to represent values of pointer, channel and
interface types.

### Concept: Value Parts

At run time, many values are stored somewhere in memory. In Go, each of
such values has a direct part, however, some of them each has one or
more indirect parts. Each value part occupies a continuous memory
segment. The indirect underlying parts of a value are referenced by its
direct part through [pointers](pointer.html).

The terminology [**value part**](value-part.html) is not defined in Go
specification. It is just used in Go 101 to make some explanations
simpler and help Go programmers understand Go types and values better.

### Concept: Value Sizes

When a value is stored in memory, the number of bytes occupied by the
direct part of the value is called the size of the value. All values of
the same type have the same value size, so the size of values of a type
is often called as the size, or value size, of the type.

We can use the `Sizeof` function in the `unsafe` standard package to get
the size of any value.

Go specification doesn't specify values size requirements for
non-numeric types. The requirements for value sizes of all kinds of
basic numeric types are listed in the article [basic Types and basic
value literals](basic-types-and-value-literals.html).

### Concept: Base Type Of A Pointer Type

For a pointer type, assume its underlying type can be denoted as `*T` in
literal, then `T` is called the base type of the pointer type.

For more about pointer types and values, please read [pointers in
Go](pointer.html).

### Concept: Fields Of A Struct Type

<div>

A struct type is composed of a collection of member variables. Each of
the member variables is called a field of the struct type. For example,
the following struct type `Book` has three fields, `author`, `title` and
`pages`.

``` line-numbers
struct {
    author string
    title  string
    pages  int
}
```

</div>

For more about struct types and values, please read [structs in
Go](struct.html).

### Concept: Signature Of Function Types

The signature of a function type is composed of the input parameter
definition list and the output result definition list of the function.

The function name and body are not parts of a function signature.
Parameter and result types are important for a function signature, but
parameter and result names are not important.

Please read [functions in Go](function.html) for more details about
function type and function values.

### Concept: Method And Method Set Of A Type

In Go, some types can have [methods](method.html). Methods can also be
called member functions. The method set of a type is composed of all the
methods of the type.

### Concept: Dynamic Type And Dynamic Value Of An Interface Value

Interface values are the values whose types are interface types.

Each interface value can box a non-interface value in it. The value
boxed in an interface value is called the dynamic value of the interface
value. The type of the dynamic value is called the dynamic type of the
interface value. An interface value boxing nothing is a zero interface
value.

An interface type can specified zero or several methods, which form the
method set of the interface type.

If the method set of a type, which is either an interface type or a
non-interface type, is the super set of the method set of an interface
type, we say the type [implements](interface.html#implementation) the
interface type.

For more about interface types and values, please read [this
article](interface.html).

### Concept: Container Types

Array, slice and map can be viewed as formal built-in container types.

Sometomes, string and channel types can also be viewed as container
types informally.

Each value of a container type has a length, either the container type
is a formal one or an informal one.

For more about formal container types and values, please read
[containers in Go](container.html).

### Concept: Key Type Of A Map Type

If the underlying type of a map type can be denoted as `map[Tkey]T`,
then `Tkey` is called the key type of the map type. `Tkey` must be a
comparable type (see below).

### Concept: Element Type Of A Container Type

<div>

The types of the elements stored in a container value must be identical.
The identicial type of the elements is called the element type of the
container type of the container value.

  - For an array type, if its underlying type is `[N]T`, then its
    element type is `T`.
  - For a slice type, if its underlying type is `[]T`, then its element
    type is `T`.
  - For a map type, if its underlying type is `map[Tkey]T`, then its
    element type is `T`.
  - For a channel type, if its underlying type is `chan T`, `chan<- T`
    or `<-chan T`, then its element type is `T`.
  - The element type of any string type is always `byte` (a.k.a.
    `uint8`).

</div>

### Concept: Directions Of Channel Types

<div>

Channel values can be viewed as synchronized first-in-first-out (FIFO)
queues. Channel types and values have directions.

  - A channel value which is both sendable and receivable is called a
    bidirectional channel. Its type is called a bidirectional channel
    type. The underlying types pf bidirectional channel types are
    denoted as `chan T` in literal.
  - A channel value which is only sendable is called a send-only
    channel. Its type is called a send-only channel type. Send-only
    channel types are denoted as `chan<- T` in literal.
  - A channel value which is only receivable is called a receive-only
    channel. Its type is called a receive-only channel type.
    Receive-only channel types are denoted as `<-chan T` in literal.

</div>

For more about channel types and values, please read [channels in
Go](channel.html).

### Fact: Types Which Support Or Don't Support Comparisons

<div>

Currently (Go 1.11), following types don't support comparisons between
values of the following types (with the `==` and `!=` operators):

  - slice types
  - map types
  - function types
  - any struct type with a field whose type is uncomparable and any
    array type which element type is uncomparable.

Above listed types are called uncomparable types. All other types are
called comparable types. Compilers forbid comparing two values of
uncomparable types.

Note, some Go tools may call uncomparable types as incomparable types.

The key type of any map type must be a comparable type.

We can learn more about the detailed rules of comparisons from the
article [value conversions, assignments and comparisons in
Go](value-conversions-assignments-and-comparisons.html#comparison-rules).

</div>

### Fact: Object-Oriented Programming In Go

<div>

Go is not a full-featured object-oriented programming language, but Go
really supports some object-oriented programming styles. Please read the
following listed articles for details:

  - [methods in Go](method.html).
  - [implementations in Go](interface.html#implementation).
  - [type embedding in Go](type-embedding.html).

</div>

### Fact: Generics In Go

Up to now (v1.11), the generic functionalities in Go are limited to
built-in types and functions. Custom generics are still in draft phase
now. Please read [built-in generics in Go](generic.html) for details.

-----

The ***Go 101*** project is hosted on both [Github][20] and [Gitlab][21]). 
Welcome to improve ***Go 101*** articles by submitting corrections for all kinds 
of mistakes, such as typos, grammar errors, wording inaccuracies, description 
flaws, code bugs and broken links.

Support Go 101 by playing [Tapir's games][22].

[20]: https://github.com/go101/go101
[21]: https://gitlab.com/Go101/go101
[22]: https://www.tapirgames.com
