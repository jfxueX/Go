# [Interfaces In Go][1]

[1]: https://github.com/go101/go101/edit/master/articles/interface.html

Interface types are one special kind of types in Go. Interface kind plays 
several important roles in Go. Firstly, interface types make Go support value 
boxing. Consequently, through value boxing, reflection and polymorphism get 
supported.

The remaining of this article will explain the functionalities of interfaces in 
detail. Some interface related details will also be shown.

### What Are Interface Types?

An interface type specifies a collection of [method prototypes][2]. In other 
words, each interface type defines a method set. In fact, we can view an 
interface type as a method set. For any of the method prototype specified in an 
interface type, its name can't be the blank identifier `_`.

[2]: method.html#method-set
[3]: https://golang.org/pkg/builtin/#error

Some examples of interface types:

```go
// This is an unnamed interface type.
interface {
    About() string
}

// ReadWriter is a defined interface type.
type ReadWriter interface {
    Read(buf []byte) (n int, err error)
    Write(buf []byte) (n int, err error)
}

// Comparable is a named interface alias type.
type Comparable = interface {
    IsEqualTo(another Comparable) bool
}
```

Please note that the `error` result type in the method prototypes specified by 
the `ReadWriter` interface type is [a built-in interface type][3]. It is defined 
as

```go
type error interface {
    Error() string
}
```

In particular, an interface type with a blank method set is called a blank 
interface type. Here are some blank interface types examples:

```go
// An unnamed blank interface type.
interface{}

// Type I is a defined blank interface type.
type I interface{}
```


### The Method Set Of A Type


Each type has a [method set][4] associated with it.

  - For an interface type, its method set is the method prototype collection it 
    specifies.
  - For a non-interface type, its method set is the prototype collection of all 
    the methods (either explicit or implicit ones) declared for it.

*(About methods of non-interface types, please read [methods in Go][5] and [type 
embedding][6].)*

For convenience, the method set of a type is often also called the method set of 
any value of the type.

*Two unnamed interface types are identical if their method sets are identical.*

[4]: method.html#method-set%22
[5]: method.html
[6]: type-embedding.html

### What Are Implementations?

If the method set of an arbrtrary type `T`, `T` may be an interface type or not, 
is a super set of the method set of an interface type `I`, then we say type `T` 
implements interface `I`.

Implementations are all implicit in Go. The implementation relations are not 
needed to be specified for compilers in code explicitly. There is not an 
`implements` keyword in Go. Go compilers will check the implementation relations 
automaticlly as needed.

An interface type always implements itself. Two interface types with the same 
method set implement each other.

For example, in the following example, the method sets of stuct pointer type 
`*Book`, integer type `MyInt` and pointer type `*MyInt` all contain the method 
prototype `About() string`, so they all implement the above mentioned interface 
type `interface {About() string}`.

```go
type Book struct {
    name string
    // more other fields ...
}

func (book *Book) About() string {
    return "Book: " + book.name
}

type MyInt int

func (MyInt) About() string {
    return "I'm a custom integer value"
}
```

Note, as any method set is a super set of a blank method set, so **any type 
implements any blank interface type**. This is an important fact in Go.

The implicit implementation design makes it possible to let concrete types 
defined in other library packages, such as standard packages, passively 
implement some interface types declared in user packages. For example, if we 
define an interface type as the following one, then the type `DB` and type `Tx` 
declared in [the `database/sql` standard package][4] will both implement the 
interface type automatically, for they both have the three corresponding methods 
specified in the interface.

```go
type DatabaseStorer interface {
    Exec(query string, args ...interface{}) (Result, error)
    Prepare(query string) (*Stmt, error)
    Query(query string, args ...interface{}) (*Rows, error)
}
```

[4]: https://golang.org/pkg/database/sql/
[5]: constants-and-variables.html#assignment

### Value Boxing

In Go, if a type `T` implements an interface type `I`, then any value of type 
`T` can be implicitly converted to type `I`. In other words, any value of type 
`T` is [assignable][5] to (addressable) values of type `I`.

In such a conversion (or an assignment),

  - If type `T` is a non-interface type, then a copy of the `T` value is boxed 
    (or encapsulated) into the result (or destination) `I` value.  The time 
    complexity of the copy is `O(n)`, where `n` is the size of copied `T` value.
  - If type `T` is also an interface type, then a copy of the value boxed in the 
    `T` value is boxed (or encapsulated) into the result (or destination) `I` value. 
    The standard Go compiler makes an optimazation here, so the time complexity of 
    the copy is `O(1)`, instead of `O(n)`.

The type infomation of the boxed value is also stored in the result interface 
value.

When a value is boxed in an interface value, the value is called the ***dynamic 
value*** of the interface value. The type of the dynamic value is called the 
***dynamic type*** of the interface value.

The dynamic value of an interface value must be a non-interface value.  The 
dynamic value itself is immutable. We can replace the dynamic value of an 
interface value with another dynamic value, but there are no (safe) ways to 
modify any part of the dynamic value.

Dynamic values and dynamic types are also called concrete values and concrete 
types sometimes.

In Go, the zero values of any interface type are represented by the predeclared 
`nil` identifier. Nothing is boxed in a nil interface value.  Assigning an 
untyped `nil` to an interface value will clear the dynamic value boxed in the 
interface value.

*(Note, the zero values of many non-interface types in Go are also represented 
by `nil` in Go. Non-interface nil values can also be boxed in interface values. 
Such interface values are not nil interface values.)*

As any type implements any blank interface types, so any non-interface value can 
be boxed in (or assigned to) a blank interface value. For this reason, blank 
interface types can be viewed as the `any` type in many other languages.

When an untyped value (except untype `nil` values) is assigned to a blank 
interface value, the untyped value will be first converted to its default type. 
(Or, we can think the untyped value is deduced as a value of its default type.)

Let's view an example which demonstrates some assignments with interface values 
as the destinations.

```go
package main

import "fmt"

type Aboutable interface {
    About() string
}

// Type *Book implements Aboutable.
type Book struct {
    name string
}
func (book *Book) About() string {
    return "Book: " + book.name
}

func main() {
    // A *Book value is boxed into an Aboutable value.
    var a Aboutable = &Book{"Go 101"}
    fmt.Println(a) // &{Go 101}

    // i is a blank interface value.
    var i interface{} = &Book{"Rust 101"}
    fmt.Println(i) // &{Rust 101}

    // Aboutable implements interface{}.
    i = a
    fmt.Println(i) // &{Go 101}
}
```

Please note, the prototype of the `fmt.Println` function is

```go
func Println(a ...interface{}) (n int, err error)
```

This is why a `fmt.Println` function call can take arguments of any types.

The following is another example which shows how a blank interface value is used 
to box values of any non-interface type.

```go
package main

import "fmt"

func main() {
    var i interface{}
    i = []int{1, 2, 3}
    fmt.Println(i) // [1 2 3]
    i = map[string]int{"Go": 2012}
    fmt.Println(i) // map[Go:2012]
    i = true
    fmt.Println(i) // true
    i = 1
    fmt.Println(i) // 1
    i = "abc"
    fmt.Println(i) // abc

    // Clear the boxed value in interface value i.
    i = nil
    fmt.Println(i) // <nil>
}
```

As above has mentioned, the information of the dynamic type of an interface 
value is also stored in (more accurately, referenced by) the interface value. 
The dynamic type information is essential for implementing two important 
features in Go.

1.  The dynamic type information includes a method table which stores
    all the corresponding methods specified by the interface type and
    declared for the dynamic type of the interface type. This table is
    the key to implement [polymorphism](#polymorphism) in Go.
2.  The dynamic type information also includes all sorts of other information 
    about the dynamic type, such as what [kind][6] the dynamic type belongs to and 
    the method and field lists of the dynamic type, etc.  The information is the key 
    to implement [reflection](#reflection) in Go.

At run time, the information items of all types are stored in a global zone. 
Each interface value just holds a reference to one type information item. Each 
non-interface to interface type implementation relation corresponds to one 
global method table. For the standard Go compiler/runtime, a method table will 
only be created as needed.

[6]: type-system-overview.html#type-kinds

### Polymorphism

Polymorphism is one key functionality provided by interfaces, and it is an 
important feature of Go.

When a non-interface value `t` of a type `T` is boxed in an interface value `i` 
of type `I`, calling a method specified by the interface type `I` on the 
interface value `i` will call the corresponding method declared for the non-
interface type `T` on the non-interface value `t` actually. In other words, 
calling method `i.m` will call method `t.m` actually. With different values 
boxed into the interface value, the interface value behaves differently. This is 
called polymorphism.

The method table stored in an interface value is a slice. Method lookup in the 
table is just a slice element indexing.

*(Note, calling methods on a nil interface value will panic at run time, for 
there are no available declared methods can be called.)*


An example:

```go
package main

import "fmt"

type Filter interface {
    About() string
    Process([]int) []int
}

// A UniqueFilter will remove duplicated numbers.
type UniqueFilter struct{}
func (UniqueFilter) About() string {
    return "remove duplicated numbers"
}
func (UniqueFilter) Process(inputs []int) []int {
    outs := make([]int, 0, len(inputs))
    pusheds := make(map[int]bool)
    for _, n := range inputs {
        if !pusheds[n] {
            pusheds[n] = true
            outs = append(outs, n)
        }
    }
    return outs
}

// A MultipleFilter will only keep numbers which are
// multiples of the MultipleFilter as an int value.
type MultipleFilter int
func (mf MultipleFilter) About() string {
    return fmt.Sprintf("keep multiples of %v", mf)
}
func (mf MultipleFilter) Process(inputs []int) []int {
    var outs = make([]int, 0, len(inputs))
    for _, n := range inputs {
        if n % int(mf) == 0 {
            outs = append(outs, n)
        }
    }
    return outs
}

// With the help of polymorphism, only one "filteAndPrint"
// function is needed.
func filteAndPrint(fltr Filter, unfiltered []int) []int {
    // Call the methods of "fltr" will call the methods
    // of the value boxed in "fltr" actually.
    filtered := fltr.Process(unfiltered)
    fmt.Println(fltr.About() + ":\n\t", filtered)
    return filtered
}

func main() {
    numbers := []int{12, 7, 21, 12, 12, 26, 25, 21, 30}
    fmt.Println("before filtering:\n\t", numbers)

    // Three non-interface values are boxed into three Filter
    // interface slice element values.
    filters := []Filter{
        UniqueFilter{},
        MultipleFilter(2),
        MultipleFilter(3),
    }

    // Each slice element will be assigned to the local variable
    // "fltr" (of interface type Filter) one by one. The value
    // boxed in each element will also be copied into "fltr".
    for _, fltr := range filters {
        numbers = filteAndPrint(fltr, numbers)
    }
}
```

The output:

```
before filtering:
     [12 7 21 12 12 26 25 21 30]
remove diplicated numbers:
     [12 7 21 26 25 30]
keep multiples of 2:
     [12 26 30]
keep multiples of 3:
     [12 30]
```

In the above example, polymorphism makes it unnecessary to write one 
`filteAndPrint` function for each filter types.

Besides the above benifit, polymorphism also makes it possible for package 
developers to write exported functions and methods which accept any user defined 
types, as long as the user types implement the interface types of the parameters 
of the exported functions and methods.

In fact, polymorphism is not an essential feature for a language. There are 
alternative ways to achieve the same job, such as callback functions. But the 
polymorphism way is cleaner and more elegant.

### Reflection

The dynamic type information stored in an interface value can be used to inspect 
and manipulate the values referenced by the dynamic value which is boxed in the 
interface value. This is called reflection in programming.

Currently (v1.11), Go doesn't support generic for custom functions and types. 
Reflection partially remedies the inconveniences caused by lacking of generic.

This article will not explain the functionalities provided by [the `reflect` 
standard package][7]. Please read [reflections in Go][8] to get how to use this 
package. Below will only introduce the built-in reflection functionalities in 
Go. In Go, built-in reflections are achieved with type assertion and type 
switch.

[7]: https://golang.org/pkg/reflect/
[8]: reflection.html

#### Type Assertion

There are four kinds of interface value involved value conversion cases in Go:

1.  convert a non-interface value to an interface value, where the type
    of the non-interface value must implement the type of the interface
    value.
2.  convert an interface value to an interface value, where the type of
    the source interface value must implement the type of the
    destination interface value.
3.  convert an interface value to a non-interface value, where the type
    of the non-interface value may or may not implement the type of the
    interface value.
4.  convert an interface value to an interface value, where the type of
    the source interface value may or may not implement the type of the
    destination interface value.

Above has explained the first two kinds of cases. The two both require the 
source value type must implement the destination interface type. The 
convertibility for the first two are verified at compile time.

Here will explain the later two kinds of cases. The convertibility for the later 
two are verified and achieved at run time, by using a syntax called ***type 
assertion***. In fact, the syntax also applies to the second kind of 
conversions.

The form of a type assertion expression is `i.(T)`, where `i` is an interface 
value and `T` is

  - either a non-interface type which must implement the type of `i`,
  - or an arbrtrary interface type.

In a type assertion `i.(T)`, `i` is called the asserted value and `T` is called 
the asserted type. A type assertion would succeed or fail.

  - If `i` is a nil interface value, then the assertion will always
    fail.
  - If `T` is a non-interface type, then the intention of the type
    assertions is to get (a copy of) the dynamic value boxed in an
    interface value. `T` and the dynamic type of `i` must be identical
    to make the assertion succeed. Each of such conversion can be viewed
    as a value unboxing attempt.
  - If `T` is an interface type, then the intention of the type
    assertion is to make the dynamic value boxed in an interface value
    also boxed in another interface value of type `T`. The dynamic value
    of `i` must be convertible to interface type `T` to make the
    assertion succeed. In other words, the dynamic type of `i` must
    implement the interface type `T` to make the assertion succeed.

For most scenarios, a type assertion results one value of the asserted type and 
is viewed as a single-valued expression. However, when a type assertion is used 
as the only source value expression in an assignment, it can be viewed as a 
multi-valued expression and result a second optional untyped boolean value, 
which indicates whether or not the type assersion succeeds.

If a type assersion fails, the first return result must be a zero value of the 
asserted type, and the second return result (assume it is present) is an untyped 
boolean value `false`. If it succeeds, then the first return result is a copy of 
the dynamic value of the asserted interface value, and the second return result 
(assume it is present) is an untyped boolean value `true`.

If a type assersion fails and the type assertion is used as a single-valued 
expression (the second return result is absent), a panic will occur.

An example (`T` is a non-interface type):

```go
package main

import "fmt"

func main() {
    // Compiler will deduce the type of 123 as int.
    var x interface{} = 123

    // Case 1:
    n, ok := x.(int)
    fmt.Println(n, ok) // 123 true
    n = x.(int)
    fmt.Println(n) // 123

    // Case 2:
    a, ok := x.(float64)
    fmt.Println(a, ok) // 0 false

    // Case 3:
    a = x.(float64) // will panic
}
```

Another example (`T` is an interface type):

```go
package main

import "fmt"

type Writer interface {
    Write(buf []byte) (int, error)
}

type DummyWriter struct{}
func (DummyWriter) Write(buf []byte) (int, error) {
    return len(buf), nil
}

func main() {
    var x interface{} = DummyWriter{}
    var y interface{} = "abc" // dynamic type is "string"
    var w Writer
    var ok bool

    // DummyWriter implements both Writer and interface{}.
    w, ok = x.(Writer)
    fmt.Println(w, ok) // {} true
    x, ok = w.(interface{})
    fmt.Println(x, ok) // {} true

    // The dynamic type of y is "string", which doesn't
    // implement Writer.
    w, ok = y.(Writer)
    fmt.Println(w, ok) // &ltnil> false
    w = y.(Writer)     // will panic
}
```

In fact, the method call of `i.m(...)` on an interface value `i` which dynamic 
type is `T` is equivalent to the method call of `i.(T).m(...)`.


#### Type Switch Control Flow Block

The type switch syntax may be the weirdest syntax in Go. It can be viewed as the 
enhanced version of type assertion. A type switch block looks like:

```go
switch aSimpleStatement; v := x.(type) {
case TypeA:
    ...
case TypeB, TypeC:
    ...
case nil:
    ...
default:
    ...
}
```

The `aSimpleStatement;` portion is optional in a type switch code block.

The `case` keywords in a type switch block can be followed by a `nil` 
identifier, one or more type names and type literals. None of such items can 
follow more than one `case` keywords at the same time in the same type switch 
block.

If the type denoted by a type name or type literal following a `case` keyword in 
a type switch block is not an interface type, then it must implement the 
interface type of the interface value followed by `.(type)`.

Here is an example of type switch:

```go
package main

import "fmt"

func main() {
    values := []interface{}{
        456, "abc", true, 0.33, int32(789),
        []int{1, 2, 3}, map[int]bool{}, nil,
    }
    for _, x := range values {
        // Here, v is declared once, but it denotes
        // different varialbes in different branches.
        switch v := x.(type) {
        case []int: // a type literal
            // The type of v is []int.
            fmt.Println("int slice:", v)
        case string: // one type name
            // The type of v is string.
            fmt.Println("string:", v)
        case int, float64, int32: // multiple type names
            // The type of v is always same as x.
            // In this example, it is interface{}.
            fmt.Println("number:", v)
        case nil:
            // The type of v is always same as x.
            // In this example, it is interface{}.
            fmt.Println(v)
        default:
            // The type of v is always same as x.
            // In this example, it is interface{}.
            fmt.Println("others:", v)
        }
        // Note, each variable denoted by v in the
        // last three branches is a copy of x.
    }
}
```

The output:

``` output
number: 456
string: abc
others: true
number: 0.33
number: 789
int slice: [1 2 3]
others: map[]
<nil>
```

The above example is equivalent to the following one:

```go
package main

import "fmt"

func main() {
    values := []interface{}{
        456, "abc", true, 0.33, int32(789),
        []int{1, 2, 3}, map[int]bool{}, nil,
    }
    for _, x := range values {
        if v, ok := x.([]int); ok {
            fmt.Println("int slice:", v)
        } else if v, ok := x.(string); ok {
            fmt.Println("string:", v)
        } else if x == nil {
            v := x
            fmt.Println(v)
        } else {
            _, isInt := x.(int)
            _, isFloat64 := x.(float64)
            _, isInt32 := x.(int32)
            if isInt || isFloat64 || isInt32 {
                v := x
                fmt.Println("number:", v)
            } else {
                v := x
                fmt.Println("others:", v)
            }
        }
    }
}
```

Some details about type switch:

  - unlike general `switch-case` blocks, `fallthrough` statements can't
    be used within branch blocks of a type switch block.
  - like general `switch-case` blocks, if the `default` branch is
    present, it can be the last branch, the first branch, or a middle
    branch.
  - if we don't care about the assertion value in a type switch, we can
    use the short form `switch x.(type) {...}` instead.
  - like general `switch-case` blocks, zero or more branches can be
    present in a type switch block. A type switch block without
    branches, `switch a.(type) {}`, is a no-op at run time.

### More About Interfaces In Go

#### Interface Type Embedding

In Go, an interface type can embed another named interface type. The final 
effect is the same as unfolding the method prototypes specified by the embedded 
interface type into the definition body of the embedding interface type. For 
example, given the following two named interface types:

```go
type Ia interface {
    fa()
}

type Ib interface {
    fb()
}
```

then the following interface type:

```go
interface {
    Ia
    Ib
}
```

has the identical method set as `Ic`:

```go
type Ic interface {
    fa()
    fb()
}
```

Up to now (Go 1.11), if two interface types both specify a method prototype with 
the same name, then they can't be embedded as the same time in the same third 
interface type, even if the two method prototypes are identical. For example, 
the following two interafce types are both illegal:

```go
interface {
    Ia
    Ic
}

interface {
    Ib
    Ic
}
```

An interface type can't embed itself or any other interface types that embeds 
the interface type, recursively.

#### Interface Values Involved Comparisons

There are two cases of interface values involved comparisons:

1.  comparisons between a non-interface value and an interface value.
2.  comparisons between two interface values.

For the first case, the type of the non-interface value must implement the type 
(assume it is `I`) of the interface value, so the non-interface value can be 
converted to (boxed into) an interface value of `I`. This means a comparison 
between a non-interface value and an interface value can be translated to a 
comparison between two interface values. So below only comparisons between two 
interface values will be explained.

Comparing two interface values is comparing their respective dynamic types and 
dynamic values actually.

The steps of comparing two interface values (with the `==` operator):

1.  if one of the two interface values is a nil interface value, then
    the comparison result is whether or not the other interface value is
    also a nil interface value.
2.  if the dynamic types of the two interface values are two different
    types, then the comparison result is `false`.
3.  for the case of the dynamic types of the two interface values are
    the same type,
      - if the same dynamic type is an [uncomparable type][9], a panic will occur.
      - otherwise, the comparison result is the result of comparing the
        dynamic values of the two interface values.

In short, two interface values are equal only if their respective dynamic types 
and their respective dynamic values are both equal, and their dynamic values are 
comparable.

[9]: value-conversions-assignments-and-comparisons.html#comparison-rules

By the rules, two interface values which dynamic values are both `nil` may be 
not equal. An example:

```go
package main

import "fmt"

func main() {
    var a, b, c interface{} = "abc", 123, "a"+"b"+"c"
    fmt.Println(a == b) // a case of step 3. Print "false".
    fmt.Println(a == c) // a case of step 3. Print "true".

    var x *int = nil
    var y *bool = nil
    var ix, iy interface{} = x, y
    var i interface{} = nil
    fmt.Println(ix == iy) // false
    fmt.Println(ix == i)  // false
    fmt.Println(ix == i)  // false

    var s []int = nil // []int is an uncomparable type
    i = s
    fmt.Println(i == nil) // a case of step 1. Print "false".
    fmt.Println(i == i)   // a case of step 2. Will panic.
}
```

</div>

#### The Internal Structure Of Interface Values

For the offcial Go compiler/runtime, blank interface values and non-blank 
interface values are represented with two different internal structures. Please 
read [value parts][10] for details.

#### Pointer Dynamic Value vs. Non-Pointer Dynamic Value

The offcial Go compiler/runtime makes an optimization which makes that boxing 
pointer values into interface values is more efficient than boxing non-pointer 
values. For [small size values][11], the efficiency differences are small, but 
for large size values, the differences may be not small. For the same 
optimization, type assertions with a pointer type are also more efficient than 
type assertions with the base type of the pointer type if the base type is a 
large size type.

So please try to avoid boxing large size values, box their pointers instead.

*(BTW, for the standard Go compiler and runtime, types other than array and 
struct types are all small size types. Struct types with very few fields are 
also viewed as small size types.)*

[10]: value-part.html#interface-structure
[11]: value-copy-cost.html

#### Values Of `[]T` Can't Be Directly Converted To `[]I` Even If Type `T` Implements Interface Type `I`.

For example, sometimes, we may need to convert a `[]string` value to 
`[]interface` type. Unlike some other languages, there is no direct ways to make 
the conversion. We must make the conversion manually in a loop:

```go
package main

import "fmt"

func main() {
    words := []string{
        "Go", "is", "a", "high",
        "efficient", "language.",
    }

    // The prototype of fmt.Println function is
    // func Println(a ...interface{}) (n int, err error).
    // So words... can't be passed to it as the argument.

    // fmt.Println(words...) // not compile

    // Convert the []string value to []interface{}.
    iw := make([]interface{}, 0, len(words))
    for _, w := range words {
        iw = append(iw, w)
    }
    fmt.Println(iw...) // compiles okay
}
```

#### Each Method Specified In An Interface Type Corresponds To An Implicit Function

For each method with name `m` in the method set defined by an interface type 
`I`, compilers will implicitly declare a function named `I.m`, which has one 
more input parameter, of type `I`, than method `m`. The extra parameter is the 
first input parameter of function `I.m`. A call to the function `I.m(i, ...)` is 
equivalent to the method call `i.m(...)`.

An example:

```go
package main

import "fmt"

type I interface {
    m(int)bool
}

type T string
func (t T) m(n int) bool {
    return len(t) > n
}

func main() {
    var i I = T("gopher")
    fmt.Println (i.m(5))                         // true
    fmt.Println (I.m(i, 5))                      // true
    fmt.Println (interface {m(int) bool}.m(i, 5)) // true

    // The following lines compile okay, but will panic at run time.
    I(nil).m(5)
    I.m(nil, 5)
    interface {m(int) bool}.m(nil, 5)
}
```
