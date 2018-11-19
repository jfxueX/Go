[Type Embedding][1]
===================

[1]: https://go101.org/article/type-embedding.html
[2]: ../structs/go101-structs-in-go.md


* [What Does Type Embedding Look Like?](#what-does-type-embedding-look-like)
* [Which Types Can be Embedded?](#which-types-can-be-embedded)
* [What Is The Meaningfulness Of Type Embedding?](#what-is-the-meaningfulness-of-type-embedding)
* [Does The Embedding Type Obtain The Fields And Methods Of The Embedded Types?](#does-the-embedding-type-obtain-the-fields-and-methods-of-the-embedded-types)
* [Shorthands Of Selectors](#shorthands-of-selectors)
* [Selector Shadowing And Colliding](#selector-shadowing-and-colliding)
* [Implicit Methods For Embedding Types](#implicit-methods-for-embedding-types)
* [Interface Types Embed Interface Types](#interface-types-embed-interface-types)
* [An Interesting Type Embedding Example](#an-interesting-type-embedding-example)


From the article [structs in Go][2], we know that a struct type can have many 
fields. Each of the fields is composed of one field name and one field type. In 
fact, sometimes, a struct field can be composed of one field type only. Such 
fields are called embedded fields.  Some articles also call embedded fields as 
anonymous fields.

This article will explain the intention of type embedding and all kinds of 
details in type embedding.

### What Does Type Embedding Look Like?

Here is an example of type embedding:

```go
package main

import "net/http"

func main() {
    type P = *bool
    var x struct {
        string // a defined non-pointer type
        error  // a defined interface type
        *int   // an unnamed pointer type
        P      // an alias of an unnamed pointer type

        http.Header // a defined map type
    }
    x.string = "Go"
    x.error = nil
    x.int = new(int)
    x.P = new(bool)
    x.Header = http.Header{}
}
```

In the above example, five types are embedded in the struct type. Each type 
embedding forms an embedded field.

Embedded fields are also called as anonymous fields. However, in fact, each 
embedded field has a name specified implicitly. The [unqualified][3] type name 
of an embedded field acts as the name of the field. For example, the names of 
the five embedded fields in the above examples are `string`, `error`, `int`, 
`P`, and `Header`, respectively.

[3]: packages-and-imports.html#import
[4]: https://golang.org/ref/spec#Struct_types
[5]: https://github.com/golang/go/issues/22005
[6]: go101-type-system-overview.md#concept-named-types-vs-unnamed-types

### Which Types Can be Embedded?

The current Go specification (v1.11) [says][4]

An embedded field must be specified as a type name T or as a pointer to a non-
interface type name `*T`, and T itself may not be a pointer type.

The above description is accurate before Go 1.09. However, with the introduction 
of type aliases in Go 1.09, the description [becomes a little outdated and 
inaccurate][5]. For example, the description doesn't include the case of the `P` 
field in the example in the last section.

The following descriptions are more accurate on which types can and can't be 
embedded.

A type can be embeded if it satisfies any of the following three conditions.

1.  The type is a [named type][6] which is not a pointer type.
2.  The type is an unnamed pointer type which based type is a named type but 
    neither a pointer nor interface type.
3.  The type is an alias type of a type of the second case.

In other words,

the following types can't be embedded.

1.  [Defined][7] pointer types.
2.  Unnamed non-pointer types.
3.  Pointer types whose base types are either interface or poiner types.

The above rules for which types can be embedded are unnecessarily restricted and 
complicated, which makes it some hard to describe them clearly in brief. It 
looks they are intended to avoid types without methods being embedded. However, 
at the same time, they also already allow many types without methods to be 
embedded, such as built-in basic types and aliases of many unnamed types. There 
is [a proposal][8] trying to simplify the rules but it was declined. The rule 
specified in the proposal is quite simple and doesn't do any harm: **if a type 
can provide an unqualified name, then the type can be embedded**.

[7]: type-system-overview.html#non-defined-type
[8]: https://github.com/golang/go/issues/24062

Following lists some example types which can and can't be embedded (in struct 
types):

```go
type Encoder interface {Encode([]byte) []byte}
type Person struct {name string; age int}
type Alias = struct {name string; age int}
type AliasPtr = *struct {name string; age int}
type IntPtr *int

// These types can be embedded in a struct type.
Encoder
Person
*Person
Alias
*Alias
AliasPtr
int
*int

// These types can't be embedded.
*Encoder         // base type is an interface type
*AliasPtr        // base type is a pointer type
IntPtr           // defined pointer type
*IntPtr          // base type is a pointer type
struct {age int} // unnamed non-pointer type
map[string]int   // unnamed non-pointer type
[]int64          // unnamed non-pointer type
func()           // unnamed non-pointer type
```

No two fields are allowed to have the same name in a struct, there are not 
exceptions for annonymous struct fields. By the embedded field naming rules, an 
unnamed pointer type can't be embedded along with its base type in the same 
struct type. For example, `int` and `*int` can't be embedded in the same struct 
type.

A struct type can't embed itself or its alias types, recursively.

Generally, it is only meaningful to embed types who have fields or methods (the 
following sections will explain why), though some types without any field and 
method can also be embedded.

### What Is The Meaningfulness Of Type Embedding?

The main purpose of type embedding is to extend the functionalities of the 
embedded types into the embedding type, so that we don't need to reimplement the 
functionalities of the embeded types for the embedding type.

Many other object-oriented programming languages use inheritance to achieve the 
same goal of type embedding. Both mechanisms have their own [benefits and 
drawbacks][9].  Here, this article will not discuss which one is better. We 
should just know Go chose the type embedding mechanism, and there is a big 
difference between the two:

  - If a type `T` inherits another type, then type `T` obtains the abilities of 
    the other type. At the same time, each value of type `T` can also be viewed 
    as a value of the other type.
  - If a type `T` embeds another type, then type other type becomes a part of 
    type `T`, and type `T` obtains the abilities of the other type, but none 
    values of type `T` can be viewed as values of the other type.

[9]: https://en.wikipedia.org/wiki/Composition_over_inheritance

Here is an example to show how an embedding type extends the functionalities of 
the embedded type.

```go
package main

import "fmt"

type Person struct {
    Name string
    Age  int
}
func (p Person) PrintName() {
    fmt.Println("Name:", p.Name)
}
func (p *Person) SetAge(age int) {
    p.Age = age
}

type Singer struct {
    Person // extends Person by embedding it
    works  []string
}

func main() {
    var gaga = Singer{Person: Person{"Gaga", 30}}
    gaga.PrintName() // Name: Gaga
    gaga.Name = "Lady Gaga"
    (&gaga).SetAge(31)
    (&gaga).PrintName()   // Name: Lady Gaga
    fmt.Println(gaga.Age) // 31
}
```

From the above example, it looks that, after embedding type `Person`, the type 
`Singer` obtains (inherits) all methods and fields of type `Person`, and type 
`*Singer` obtains (inherits) all methods of type `*Person`. Are the conclusions 
right? The following sections will answer this question.

Please note that, a `Singer` value is not a `Person` value, the following code 
doesn't compile:

```go
var gaga = Singer{}
var _ Person = gaga
```

### Does The Embedding Type Obtain The Fields And Methods Of The Embedded Types?

Let's list all the fields and methods of type `Singer` and the methods of type 
`*Singer` used in the last example by using reflection functionalities provided 
in the `reflect` standard package:

```go
package main

import (
    "fmt"
    "reflect"
)

... // the types declared in the last example

func main() {
    t := reflect.TypeOf(Singer{}) // the Singer type
    fmt.Println(t, "has", t.NumField(), "fields:")
    for i := 0; i < t.NumField(); i++ {
        fmt.Print("   field#", i, ": ", t.Field(i).Name, "\n")
    }
    fmt.Println(t, "has", t.NumMethod(), "methods:")
    for i := 0; i < t.NumMethod(); i++ {
        fmt.Print("   method#", i, ": ", t.Method(i).Name, "\n")
    }
    
    pt := reflect.TypeOf(&Singer{}) // the *Singer type
    fmt.Println(pt, "has", pt.NumMethod(), "methods:")
    for i := 0; i < pt.NumMethod(); i++ {
        fmt.Print("   method#", i, ": ", pt.Method(i).Name, "\n")
    }
}
```

The result:

```bash
main.Singer has 2 fields:
   field#0: Person
   field#1: works
main.Singer has 1 methods:
   method#0: PrintName
*main.Singer has 2 methods:
   method#0: PrintName
   method#1: SetAge
```

From the result, we know that the type `Singer` really owns a `PrintName` 
method, and the type `*Singer` really owns two methods, `PrintName` and 
`SetAge`. But the type `Singer`doesn't own a `Name` field. Then why is the 
selector expression `gaga.Name` legal for a `Singer` value `gaga`? Please read 
the next section to get the reason.

### Shorthands Of Selectors

From the articles [structs in Go][10] and [methods in Go][11], we have learned 
that, for a value `x`, `x.y` is called a selector, where `y` is either a field 
name or a method name. If `y` is a field name, then `x` must be a struct value 
or a struct pointer value.  A selector is an expresssion, which represents a 
value. If the selector `x.y` denotes a field, it may also has its own fields (if 
`x.y` is a struct value) and methods. Such as `x.y.z`, where `z` can also be 
either a field name or a method name.

[10]: ../structs/go101-structs-in-go.md
[11]: ../methods/go101-methods-in-go.md

There is a syntactic sugar in Go, (without considering selector colliding and 
shadowing explained in a later section), ***if a middle name in a seletor 
corresponds to an embedded field, then that name can be omitted from the 
selector***. This is why embedded types are also called anonymous fields.

For example:

```go
package main

type A struct {
    x int
}
func (a A) MethodA() {}

type B struct {
    A
}
type C struct {
    B
}
    
func main() {
    var c C
    
    // The following 4 lines are equivalent.
    _ = c.B.A.x
    _ = c.B.x
    _ = c.A.x
    _ = c.x // x is called a promoted field of type C
    
    // The following 4 lines are equivalent.
    c.B.A.MethodA()
    c.B.MethodA()
    c.A.MethodA()
    c.MethodA()
}
```

This is why the expression `gaga.Name` is legal in the example in the last 
section. For it is just the shorthand of `gaga.Person.Name`. `Name` is called a 
promoted field of type `Singer`.

As any embedding type must be a struct type, the article [structs in Go][12] has 
mentioned that the field of an addressable struct value can be accessed through 
the pointers of the struct value. So the following code is also legal in Go.

[12]: ../structs/go101-structs-in-go.md
[13]: ../methods/go101-methods-in-go.md#call

```go
func main() {
    var c C
    pc = &c
    
    // The following 4 lines are equivalent.
    fmt.Println(pc.B.A.x)
    fmt.Println(pc.B.x)
    fmt.Println(pc.A.x)
    fmt.Println(pc.x)
    
    // The following 4 lines are equivalent.
    pc.B.A.MethodA()
    pc.B.MethodA()
    pc.A.MethodA()
    pc.MethodA()
}
```

Similarly, the selector `gaga.PrintName` can be viewed as a shorthand of 
`gaga.Person.PrintName`. But, it is also okay if we think it is not a shorthand. 
After all, the type `Singer` really has a `PrintName` method, though the method 
is declared implicitly (please read the next next section for details). For the 
similar reason, the selector `(&gaga).PrintName` and `(&gaga).SetAge` can also 
be viewed as, or not as, shorthands of `(&gaga.Person).PrintName` and 
`(&gaga.Person).SetAge`.

Note, we can also use the selector `gaga.SetAge`, but only if `gaga` is an 
addressable value of type `Singer`. It is just a sugar of `(&gaga).SetAge`. 
Please read [method calls][13] for details.

In the above examples, `c.B.A.x` is called the full form of selectors `c.x`, 
`c.B.x` and `c.A.x`. Similarly, `c.B.A.MethodA` is called the full form of 
selectors `c.MethodA`, `c.B.MethodA` and `c.A.MethodA`.

If every middle name in the full form of a selector corresponds to an embedded 
field, then the number of middle names in the selector is called the depth of 
the selector. For example, the depth of the selector `c.MethodA` used in an 
above example is *2*, for the full form of the selector is `c.B.A.MethodA`.

### Selector Shadowing And Colliding

For a value `x`, it is possible that many of its full-form selectors have the 
same last item `y` (which is either a explicitly declared field or method) and 
every middle names of these selectors represents an embedded field. For such 
cases,

  - only the full-form selector with the shallowest depth can be shortened as 
    `x.y`. In other words, `x.y` denotes the full-form selector with the 
    shallowest depth. Other full-form selectors are **shadowed** by the one with 
    the shallowest depth.
  - if there are more than one full-form selectors with the shallowest depth, 
    then none of those full-form selectors can be shortened as `x.y`. We say 
    those full-form selectors with the shallowest depth are **colliding** with 
    each other.

If a method selector is shadowed by another method selector, and the two 
corresponding method signatures are identical, we say the first method is 
overridden by the other one.

For example, assume `A`, `B` and `C` are three defined types.

```go
type A struct {
    x string
}
func (A) y(int) bool {
    return false
}

type B struct {
    y bool
}
func (B) x(string) {}

type C struct {
    B
}
```

The following code doesn't compile. The reasons is selector `v1.A.x` collides 
with `v1.B.x` are collided with each other, so both of them can't be shortened 
to `v1.x`. The same sotuation is for selector `v1.A.y` and `v1.B.y`.

```go
var v1 struct {
    A
    B
}

func f1() {
    _ = v1.x
    _ = v1.y
}
```

The following code compiles okay. The selector `v2.C.B.x` is shadowed by 
`v2.A.x`, so the selector `v2.x` is a shortened form of `v2.A.x` actually. For 
the same reason, the selector `v2.y` is a shortened form of `v2.A.y`, not of 
`v2.C.B.y`.

```go
var v2 struct {
    A
    C
}

func f2() {
    fmt.Printf("%T \n", v2.x) // string
    fmt.Printf("%T \n", v2.y) // func(int) bool
}
```

### Implicit Methods For Embedding Types

As above has mentioned, both of type `Singer` and type `*Singer` have a 
`PrintName` method each. And the type `*Singer` also has a `SetAge` method. 
However, we never explicitly declare these methods for the two types. Where do 
these methods come from?

In fact,

  - for each method of an embedded type, if the selectors to that method neither 
    collide with nor are shadowed by other selectors, then compilers will 
    implicitly declare a correspoding method for the embedding struct type. And 
    consequently, compiler will also [implicitly declare a correspoding 
    method][14] for the unnamed pointer type whose base type is the embedding 
    struct type.
  - assume a struct type `S` embeds a named type `T`, for each method of type 
    `*T`, if the selectors to that method neither collide with nor are shadowed 
    by other selectors, then compilers will implicitly declare a correspoding 
    method for type `*S`.

Simply speaking,

  - type `struct{T}` obtains (inherit) all the methods of type `T`.
  - type `*struct{T}` obtains (inherit) all the methods of type `*T`.
  - both type `struct{*T}` and `*struct{*T}` also obtain (inherit) all the 
    methods of type `*T`.

Note, as [the method set of type `T` is always a subset of the method set of 
type `*T`][15], we can also say all the three types `*struct{T}`, `struct{*T}` 
and `*struct{*T}` obtain (inherit) all the methods of type `T`.

[14]: ../methods/go101-methods-in-go.md#implicit-pointer-methods
[15]: ../methods/go101-methods-in-go.md#method-set

Here are the implicitly declared methods for type `Singer` and type `*Singer`.

```go
func (s Singer) PrintName() {
    s.Person.PrintName()
}

func (s *Singer) PrintName() {
    (*s).Person.PrintName()
}

func (s *Singer) SetAge(age int) {
    (&s.Person).SetAge(age) // <=> (&((*s).Person)).SetAge(age)
}
```

From the article [methods in Go][16], we know that we can't explicitly declare 
methods for non-defined struct types and non-defined pointer types whose base 
types are non-defined struct types. But through type embedding, such non-defined 
types can also own methods.

[16]: ../methods/go101-methods-in-go.md

Here is another example to show which implicit methods are declared.

```go
package main

import "fmt"
import "reflect"

type F func(int) bool
func (f F) Validate(n int) bool {
    return f(n)
}
func (f *F) Modify(f2 F) {
    *f = f2
}

type B bool
func (b B) IsTrue() bool {
    return bool(b)
}
func (pb *B) Invert() {
    *pb = !*pb
}

type I interface {
    Load()
    Save()
}

func main() {
    PrintTypeMethods := func(t reflect.Type) {
        fmt.Println(t, "has", t.NumMethod(), "methods:")
        for i := 0; i < t.NumMethod(); i++ {
            fmt.Print("   method#", i, ": ", t.Method(i).Name, "\n")
        }
    }
    
    var s struct {
        F
        *B
        I
    }
    
    PrintTypeMethods(reflect.TypeOf(s))
    fmt.Println()
    PrintTypeMethods(reflect.TypeOf(&s))
}
```

The result:

```bash
struct { main.F; *main.B; main.I } has 5 methods:
   method#0: Invert
   method#1: IsTrue
   method#2: Load
   method#3: Save
   method#4: Validate

*struct { main.F; *main.B; main.I } has 6 methods:
   method#0: Invert
   method#1: IsTrue
   method#2: Load
   method#3: Modify
   method#4: Save
   method#5: Validate
```

If a struct type embeds a type which implements an interface type (the embedded 
type may be the interface type itself), then generally the struct type also 
implements the interface type, exception there is a method specified by the 
interface type shadowed by or colliding other methods or fields.

Please note, a type will only obtain the methods of the types its contains 
(embeds directly or indirectly). For example, in the following code,

  - type `Age` has no methods, for it doesn't embed any types.
  - type `X` has two methods, `IsOdd` and `Double`. `IsOdd` is obtained through 
    embedding type `MyInt`.
  - type `Y` has no methods, for its embedded type `Age` has not methods.
  - type `Z` has only one method, `IsOdd`. for it contains type `MyInt`.  It 
    doesn't obtain the method `Double` from type `X`, for it doesn't contain 
    type `X`.

```go
type MyInt int
func (mi MyInt) IsOdd() bool {
    return mi%2 == 1
}

type Age MyInt

type X struct {
    MyInt
}
func (x X) Double() MyInt {
    return x.MyInt + x.MyInt
}

type Y struct {
    Age
}

type Z X
```


### Interface Types Embed Interface Types

Not only can struct types embed other types, but also can interface types. But 
interface types can only embed named interface types. Please read [interfaces in 
Go][16] for details.

### An Interesting Type Embedding Example

In the end, let's view an interesting example. The example program will dead 
loop and stack overflow. If you have understood the above content and 
[polymorphism][17], it is easy to understand why it will dead loop.

[16]: ../interface/go101-interfaces-in-go.md#embedding
[17]: ../interface/go101-interfaces-in-go.md#polymorphism

```go
package main

type I interface {
    m()
}

type T struct {
    I
}

func main() {
    var t T
    var i = &t
    t.I = i
    i.m() // will call t.m(), then will call i.m() again, ...
}
```

********************************************************************************

The ***Go 101*** project is hosted on both [Github][30] and [Gitlab][31]). 
Welcome to improve ***Go 101*** articles by submitting corrections for all kinds 
of mistakes, such as typos, grammar errors, wording inaccuracies, description 
flaws, code bugs and broken links.

Support Go 101 by playing [Tapir's games][32].

[30]: https://github.com/go101/go101
[31]: https://gitlab.com/Go101/go101
[32]: https://www.tapirgames.com

