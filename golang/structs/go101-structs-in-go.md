[Structs In Go][1]
==================

[1]: https://go101.org/article/struct.html

Same as C, Go also supports struct types. This article will introduce the basic 
knowledges of struct types and values in Go.

### Struct Types And Struct Type Literals

Each unnamed struct type literal starts with a `struct` keyword which is 
following by a sequence of field definitions enclosed in a `{}`. Each field 
definition is composed of a name and a type. The number of fields of a struct 
type can be zero.

The following is an unnamed struct type literal:

```go
struct {
    title  string
    author string
    pages  int
}
```

The above struct type has three fields. The types of the two fields `title` and 
`author` are both `string`. The type of the `pages` field is `int`.

Some articles also call fields as member variables.

Consecutive fields with the same type can be declared together.

```go
struct {
    title, author string
    pages         int
}
```

The size of a struct type is the sum of the sizes of all its field types plus 
the number of some padding bytes. The padding bytes are used to align the memory 
addresses of some fields. We can learn padding and memory address alignments in 
[a later article][2].

[2]: memory-layout.html

*The size of a zero-field struct type is zero.*

Each struct field can be bound with a tag when it is declared. Field tags are 
optional, the default value of each field tag is a blank string. Here is an 
example showing non-default field tags.

```go
struct {
    Title  string `json:"title"`
    Author string `json:"author,omitempty"`
    Pages  int    `json:"pages,omitempty"`
}
```

The purpose of each field tag is application dependent. In the above example, 
the field tags can help the functions in the `encoding/json` standard package to 
determine the field names in JSON texts, in the process of encoding struct 
values into JSON texts or decoding JSON texts into struct values. The functions 
in the `encoding/json` standard package will only encode and decode the exported 
struct fields, which is why the first letters of the field names in the above 
example are all upper cased.

It is not a good idea to use field tags as comments.

Raw string literals (`` `...` ``) are used more popular than interpreted string 
literals (`"..."`) for field tags in practice.

Unlike C language, Go structs don't support unions.

All above shown struct types are unnamed, or called anonymous struct types. In 
practice, named (defined or alias) struct types are used more popularly.

Only exported fields of struct types shown up in a package can be used in other 
packages by importing the package. We can view non-exported struct fileds as 
private/protected member variables.

The field tags and the order of the field declarations in a struct type matter 
for the identity of the struct type. Two unnamed struct types are identical only 
if they have the same sequence of field declarations. Two field declarations are 
identical only if their respective names, their respective types and their 
respective tags are all identical. Please note, **two non-exported struct field 
names from different packages are always viewed as two different names.**

A struct type can't have a field of the struct type itself, either directly or 
recursively.


### Struct Value Literals And Struct Value Manipulations

In Go, the form `T{...}`, where `T` must be a type literal or a type name, is 
called as a **composite literal** and is used as the value literals of some 
kinds of types, including struct types and the container types introduced later.

Given a struct type `S` and assume it has two fields, `x int` and `y bool` in 
order, then the zero value of `S` can be represented by the following two 
variants of struct composite literal forms:

1.  `S{0, false}`. In this variant, none field names are present but all field 
    values must be present by the field declaration orders.
2.  `S{x: 0, y: false}`  
    `S{y: false, x: 0}`  
    `S{x: 0}`  
    `S{y: false}`  
    `S{}`  
    In this variant, each field item is optional to be present and the order 
    of the field items is not important. The values of the absent fields will be 
    set as the zero values of their respective types. But if a field item is 
    present, it must be presented with the `FieldName: FieldValue` form. The 
    order of the field items in this form doesn't matter. The form `S{}` is the 
    most used zero value representation of type `S`.

If `S` is a struct type imported from another package, it is recommended to use 
the second form, for the maintainer of the other package may add new fields for 
the type `S` later, which will make the uses of first forms become invalid.

Surely, we can also use the struct composite literals to represent non-zero 
struct value.

For a value `v` of type `S`, we can use `v.x` and `v.y`, which are called 
selectors, to access the field values of `v`. Later, we call the dot `.` in a 
selector as the property accessment operator.

An example:

```go
package main

import (
    "fmt"
)

type Book struct {
    title, author string
    pages         int
}

func main() {
    book := Book{"Go 101", "Tapir", 256}
    fmt.Println(book) // {Go 101 Tapir 256}

    // Create a book value with another form.
    book = Book{} // title and author are both "", pages is 0
    book = Book{author: "Tapir"} // title is "" and pages is 0
    book = Book{author: "Tapir", pages: 256, title: "Go 101"}

    // Initialize a struct value by using selectors.
    var book2 Book // <=> book2 := Book{}
    book2.author = "Tapir Liu"
    book2.pages = 300
    fmt.Println(book.pages) // 300
}
```

The last `,` in a composite literal is optional if the last item in the literal 
and the closing `}` are at the same line. Otherwise, the last `,` is required. 
(For more details, please read [line break rules in Go][3].)

[3]: line-break-rules.html

```go
var _ = Book {
    author: "Go 101",
    pages: 256,
    title: "Go 101", // here, the "," can not be ommitted.
}

// The last "," in the following line can be ommitted.
var _ = Book{author: "Go 101", pages: 256, title: "Go 101",}
```

### About Struct Value Assignments

When a struct value is assigned to another struct value, the effect is the same 
as assigning each field one by one.

```go
func f() {
    book1 := Book{pages: 300}
    book2 := Book{"Go 101", "Tapir", 256}

    book2 = book1
    // The above line is equivalent to the following three lines.
    book2.title = book1.title
    book2.author = book1.author
    book2.pages = book1.pages
}
```

Two struct values can be assigned to each other only if their types are 
identical or the types of the two struct values have the identical underlying 
type (considering field tags) and at least one of the two types is an [non-
defined type][4].

[4]: type-system-overview.html#non-defined-type


### Struct Field Addressability

The fields of an addressable struct value are also addressable. The fields of an 
unaddressable struct value are also unaddressable. The fields of unaddressable 
struct values can't be modified. All composite literals, including struct 
composite literal are unaddressable.

Example:

```go
package main

import "fmt"

func main() {
    type Book struct {
        Pages int
    }
    var book = Book{} // book is addressable
    p := &book.Pages
    *p = 123
    fmt.Println(book) // {123}

    // The following two lines fail to compile,
    // for Book{}.Pages is not addressable.
    /*
    Book{}.Pages = 123
    p = &(Book{}.Pages) // <=> p = &(Book{}.Pages)
    */
}
```

Please, the precedence of the property accessment operator `.` in a selector is 
higher than the address-taken operator `&`.

### Composite Literals Are Unaddressable But Can Be Taken Addresses

Generally, only addressable values can be taken addresses. But there is a 
syntactic sugar in Go, which allows us to take addesses on composite literals. A 
syntactic sugar is an exception in syntax to make programming convenient.

For example,

```go
package main

func main() {
    type Book struct {
        Pages int
    }
    // Book{100} is unaddressable but can be taken address.
    p := &Book{100} // <=> tmp := Book{100}; p := &tmp
    p.Pages = 200
}
```

### In Selectors, Struct Pointers Can Be Used As Struct Values

Unlike C, in Go, there is not the `->` operator for accessing struct fields 
through struct pointers. In Go, the `->` operator is also represented with the 
dot sign `.`.

For example:

```go
package main

func main() {
    type Book struct {
        pages int
    }
    book1 := &Book{100} // book1 is a struct poiner
    book2 := new(Book)  // book2 is another struct pointer
    book2.pages = book1.pages // use struct pointers as structs
}
```

### About Struct Value Comparisons

Most struct types are comparable types, except the ones who have fields of 
[uncomparable types][5].

[5]: type-system-overview.html#types-not-support-comparison

Two struct values are comparable only if they can be assigned to each other and 
their types are both comparable. In other words, two struct values can be 
compared with each other only if the (comparable) types of the two struct values 
have the identical underlying type (considering field tags) and at least one of 
the two types is non-defined.

When comparing two struct values of the same type, each pair of their 
corresponding fields will be compared. The two struct values are equal only if 
all of their corresponding fields are equal.

### About Struct Value Conversions

Values of two struct types `S1` and `S2` can be converted to each other's types, 
if `S1` and `S2` share the identical underlying type (by ignoring field tags). 
In particular if either `S1` or `S2` is a [non-defined type][6] and their 
underlying types are identical (considering field tags), then the conversions 
between the values of them can be implicit.

[6]: type-system-overview.html#non-defined-type

Given struct types `S0`, `S1`, `S2`, `S3` and `S4` in the following code 
snippet,

  - values of type `S0` can't be converted to the other four types, and vice 
    versa, for the corresponding field names are different.
  - two values of two different types among `S1`, `S2`, `S3` and `S4` can be 
    converted to each other's type.

In particular,

  - values of type `S2` can be implicitly converted to type `S3`, and vice 
    versa.
  - values of type `S2` can be implicitly converted to type `S4`, and vice 
    versa.

But,

  - values of type `S1` must be explicitly converted to type `S2`, and vice 
    versa.
  - values of type `S3` must be explicitly converted to type `S4`, and vice 
    versa.

```go
package main

type S0 struct {
    y int "foo"
    x bool
}

type S1 = struct { // S1 is a (non-defined) alias type
    x int "foo"
    y bool
}

type S2 = struct { // S2 is a (non-defined) alias type
    x int "bar"
    y bool
}

// If field tags are ignored, the underlying types of
// S3(S4) and S1 are same. If field tags are considered,
// the underlying types of S3(S4) and S1 are different.
type S3 S2 // S3 is a defined type
type S4 S3 // S4 is a defined type

var v0, v1, v2, v3, v4 = S0{}, S1{}, S2{}, S3{}, S4{}
func f() {
    v1 = S1(v2); v2 = S2(v1)
    v1 = S1(v3); v3 = S3(v1)
    v1 = S1(v4); v4 = S4(v1)
    v2 = v3; v3 = v2 // the conversions can be implicit
    v2 = v4; v4 = v2 // the conversions can be implicit
    v3 = S3(v4); v4 = S4(v3)
}
```

In fact, two struct values can be assigned (or compared) to each other only if 
one of them can implicitly converted to the type of the other.

### Annoymous Struct Types Can Be Used In Field Declarations

Annoymous struct types are allowed used as the types of the fields of another 
struct type. Annoymous struct type literals are also allowed to be used in 
composite literals.

An example:

```go
var aBook = struct {
    author struct { // the type of this field is an anonymous struct type
        firstName, lastName string
        gender              bool
    }
    title string
    pages int
}{
    author: struct {
        firstName, lastName string
        gender              bool
    }{
        firstName: "Mark",
        lastName: "Twain",
    }, // the type in the composite literal is an anonymous struct type
    title: "The Million Pound Note",
    pages: 96,
}
```

Generally, for better readibility, it is not recommended to use annoymous struct 
type literals in compisite literals.

### More About Struct Types

There are some advanced topics which are related to struct types. They will be 
explained in [type embedding][6] and [memory layouts][7] later.

[6]: type-embedding.html
[7]: memory-layout.html#size-and-padding

-----

The ***Go 101*** project is hosted on both [Github][20] and [Gitlab][21]). 
Welcome to improve ***Go 101*** articles by submitting corrections for all kinds 
of mistakes, such as typos, grammar errors, wording inaccuracies, description 
flaws, code bugs and broken links.

Support Go 101 by playing [Tapir's games][22].

[20]: https://github.com/go101/go101
[21]: https://gitlab.com/Go101/go101
[22]: https://www.tapirgames.com
