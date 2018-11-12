# [Methods In Go][1]

[1]: https://go101.org/article/method.html

Go supports some object-orient programming features. Method is one of these 
features. A method is a special function which has a receiver parameter (see 
below). This article will explain method related concepts in Go.

### Method Declarations

In Go, we can (explicitly) declare a method for type `T` and `*T`, where `T` 
must satisfiy 4 conditions:

1.  `T` must be a defined type;
2.  `T` must be defined in the same package as the method declaration;
3.  `T` must not be a pointer type;
4.  `T` must not be an interface type. (Interface types will be explained in 
    [the next article][2].)

`T` is called the receiver base types of all methods declared for both type `T` 
and `*T`. (Below will explain what are receivers.)

We can also declare methods for alias types of the `T` and `*T` types specified 
above. The effect is the same as declaring methods for the `T` and `*T` types 
themselves.

If a method is declared for a type, we can say the type has (or owns) the 
method.

From the above listed conditions, we will get the conclusion that we can never 
(explicitly) declare methods for

  - built-in basic types, such as `int` and `string`, for we can't declare 
    methods in the `builtin` standard package.
  - interface types. But an interface type can own methods. Please read
    [the next article][2] for details.
  - unnamed types except the pointer types `*T` which are described above, 
    including unnamed array, map, slice, function, channel and struct types. 
    However, if an unnamed struct type embeds other types which have methods, 
    then compiler will implicitly declare some methods for the unnamed struct 
    type and the unnamed pointer type whose base type is the unnamed struct 
    type. Please read [type embedding][3] for details.

[2]: ../interface/go101-interfaces-in-go.md
[3]: type-embedding.html

A method declaration is similar to a function declaration, but it has an extra 
parameter declaration part. The extra parameter is called a receiver parameter. 
Each method must be declared with one and only one receiver parameter. The 
receiver parameter must be enclosed in a `()` and declared between the `func` 
keyword and the method name. The type of the receiver parameter is called the 
receiver type of that method.

Here are some method declaration examples:

```go
// Age and int are two distinct types.
// We can't declare methods for int, but can for Age.
type Age int
func (age Age) LargerThan(a Age) bool {
    return age > a
}
func (age *Age) Increase() {
    *age++
}

// Receiver of custom defined function type.
type FilterFunc func(in int) bool
func (ff FilterFunc) Filte(in int) bool {
    return ff(in)
}

// Receiver of custom defined map type.
type StringSet map[string]struct{}
func (ss StringSet) Has(key string) bool {
    _, present := ss[key]
    return present
}
func (ss StringSet) Add(key string) {
    ss[key] = struct{}{}
}
func (ss StringSet) Remove(key string) {
    delete(ss, key)
}

// Receiver of custom defined struct type.
type Book struct {
    pages int
}
func (b Book) Pages() int {
    return b.pages
}
func (b *Book) SetPages(pages int) {
    b.pages = pages
}
```

From the above examples, we know that the receiver base types not only can be 
struct types, but also can be other kinds of types, such as basic types and 
container types, as long as the receiver base types satisfy the 4 conditions 
listed above.

In some other programming languages, the receiver parameter names are always the 
implicit `this`, which is not a recommended identifier for receiver parameter 
names in Go.

The receiver of type `*T` are called ***pointer receiver***, non-pointer 
receivers are called ***value receivers***. Personally, I don't recommand to 
view the terminology ***pointer*** as an oppoiste of the terminology 
***value***, for pointer values are just special values.  But, I am not against 
using the pointer receiver and value receiver terminologies here. The reason 
will be explained below.

Method names can be the blank identifier `_`. A type can have multiple methods 
with the blank identifier as names. But such methods can never be called.

Only exported methods can be called from other packages.

### Each Method Corresponds To An Implicit Function

For each method declaration, compiler will declare a corresponding implicit 
function for it. For the last two methods declared for type `Book` and type 
`*Book` in the last example in the last section, two following functions are 
implicitly declared by compiler:

```go
func Book.Pages(b Book) int {
    return b.pages // the body is the same as the Pages method
}

func (*Book).SetPages(b *Book, pages int) {
    b.pages = pages // the body is the same as the SetPages method
}
```

In each of the two implicit function declarations, the receiver parameter is 
removed from its correponding method declaration and inserted into the normal 
parameter list as the first one. The function bodies of the two implicitly 
declared functions are the same as their corresponding method bodies.

The implict method names, `Book.Pages` and `(*Book).SetPages`, are both of the 
form `TypeDenotation.MethodName`. As identifiers in Go can't contain the period 
special characters, the two implict function names are not legal identifiers, so 
the two functions can't be dclared explicitly. They can only be declared by 
compiler implicitly, but they can be called in user code:

```go
package main

import "fmt"

type Book struct {
    pages int
}
func (b Book) Pages() int {
    return b.pages
}
func (b *Book) SetPages(pages int) {
    b.pages = pages
}

func main() {
    var book Book
    // Call the two implicit decalred functions.
    (*Book).SetPages(&book, 123)
    fmt.Println(Book.Pages(book)) // 123
}
```

### Implicit Methods With Pointer Receivers

For each method declared for value receiver type `T`, a corresponding method 
with the same name will be implictly declared by compiler for type `*T`. By the 
example above, the `Pages` method is declared for type `Book`, so compiler will 
implicitly declare a method with the same name `Pages` for type `*Book`. The 
same name method only contain one line of code, which is a call to the implicit 
function `Book.Pages` introduced above.

```go
func (b *Book) Pages() int {
    return Book.Pages(*b)
}
```

This is why I don't reject to use the value receiver terminology (as the 
opposite of pointer receiver). After all, when we expliclty declare a method for 
a non-pointer type, in fact two methods are declared, the explicit one for the 
non-pointer type, the other implicit one is for the corresponding pointer type.

As the last section has mentioned, for each declared method, compiler will also 
declare a correponding implicit function for it. So for the just mentioned 
implicitly declared method, the following implicit function is declared by 
compiler.

```go
func (*Book).Pages(b *Book) int {
    return Book.Pages(*b)
}
```

In other words, for each explicitly declared method with a value receiver, two 
implicit functions and one implicit method will also be declared at the same 
time. <sup><a href="#note1">1</a></sup>

### Method Prototypes And Method Sets

A method prototype can be viewed as a [function prototype][4] without the `func` 
keyword. We can view each method declaration is composed of the `func` keyword, 
a receiver parameter declartion, a method prototype and a method (function) 
body.

[4]: function.html#prototype

For example, the method prototypes of the `Pages` and `SetPages` methods shown 
above are

```go
Pages() int
SetPages(pages int)
```

Each type has a method set. The method set of a non-interface type is composed 
of all the method prototypes of the methods declared, either explicitly or 
implicitly, for the type, except the ones whose names are the blank identifier 
`_`. (Interface types will be explained in [the next article][2].)

For example, the method sets of the `Book` type shown in the previous sections 
is

```go
Pages() int
```

and the method set of the `*Book` type is

```go
Pages() int
SetPages(pages int)
```

The order of the method prototypes in a method set is not important for the 
method set.

For a method set, if every method prototype in it is also in another method set, 
then we say the former method set is a subset of the latter one. If two method 
sets are subsets of each other, then we say the two method sets are identical.

Given a type `T`, assume it is neither a pointer type nor an interface type, for 
[the reason](#implicit-pointer-methods) mentioned above, the method set of a 
type `T` is always a subset of the method set of type `*T`. For example, the 
method set of the `Book` type shown above is a subset of the method set of the 
`*Book` type.

Please note, **non-exported method names, which start with lower-case letters, 
from different packages will be always viewed as two different method names**.
In other words, a method prototype with a non-exported method name is always 
different from any prototype from other packages.

Method sets play an important role in the polymorphism feature of Go.  About 
polymorphism, please read [the next article][2] (interfaces in Go) for details.

The method sets of the following types are always blank:

  - built-in basic types.
  - defined pointer types.
  - unnamed pointer types whose base types are interface or pointer types.
  - unnamed array, slice, map, function and channel types.

### Method Values And Method Calls

Methods are special functions in fact. When a method is declared for a type, 
then each value of the type will own an immutable member of a function type. The 
member name is the same as the method name. The function type is the same as the 
function declared with the form of the method declaration but without the 
receiver part. The member is often called a member function. The member function 
can also be called the method of its corresponding value.

A method call is just a call to such a member function. For a value `v`, its 
method `m` can be accessed with the selector form `v.m`.

An example containing some method calls:

```go
package main

import "fmt"

type Book struct {
    pages int
}

func (b Book) Pages() int {
    return b.pages
}

func (b *Book) SetPages(pages int) {
    b.pages = pages
}

func main() {
    var book Book

    fmt.Printf("%T \n", book.Pages)       // func() int
    fmt.Printf("%T \n", (&book).SetPages) // func(int)
    // &book has an implicit method.
    fmt.Printf("%T \n", (&book).Pages)    // func() int

    (&book).SetPages(123)
    fmt.Println(book.Pages()) // 123
    // Call the impliict method of &book.
    fmt.Println((&book).Pages()) // 123
}
```

In the above example, the value `book` is called the base value of both the 
receiver arguments of the `Pages` and `SetPages` method calls.

*(Different from C language, there is not the `->` operator in Go to call 
methods with pointer receivers, so `(&book)->SetPages(123)` is illegal in Go.)*

Method calls make Go programs clean and more readable. In fact, the above 
example can be cleaner. The line `(&book).SetPages(123)` in the above example 
can be simplified to `book.SetPages(123)`. But how can this happen? After all, 
the value `book` has not a method called `SetPages`. Aha, this is just a 
syntactic sugar to make progrmaming convenient. This sugar only works for 
addressable values of type `Book`.  Compiler will automitically take the 
addresses of the addressable values when they are passed as the receiver 
arguments of the `SetPages` method calls.

```go
 ...

// Function results are not addressable in Go.
func MakeBook() Book {
    return Book{}
}

func main() {
    var book Book
    book.SetPages(123) // <=> (&book).SetPages(123)
    fmt.Println(book.Pages()) // 123

    // error: function return results are not addressable.
    MakeBook().SetPages(123)
}
```

As above just mentioned, when a method is declared for a type, each value of the 
type will own a member function. Zero values are not exceptions, whether or not 
the zero values can be represented by `nil`.

Example:

```go
package main

type StringSet map[string]struct{}
func (ss StringSet) Has(key string) bool {
    _, present := ss[key] // Never panic here,
                          // even if ss is nil.
    return present
}

type Age int
func (age *Age) IsNil() bool {
    return age == nil
}
func (age *Age) Increase() {
    *age++ // If age is a nil pointer, then
           // dereferencing it will panic.
}

func main() {
    _ = (StringSet(nil)).Has   // will not panic
    _ = ((*Age)(nil)).IsNil    // will not panic
    _ = ((*Age)(nil)).Increase // will not panic

    _ = (StringSet(nil)).Has("key") // will not panic
    _ = ((*Age)(nil)).IsNil()       // will not panic

    // This line will panic. But the panic is not caused
    // by invoking the method. It is caused by the nil
    // pointer dereference within the method body.
    ((*Age)(nil)).Increase()
}
```

### Receiver Arguments Are Passed By Copy

Same as general function arguments, the receiver arguments are also passed by 
copy. So, the modifications on the [dirct part][5] of a receiver argument in a 
method call will not be reflected to the outside of the method.

[5]: value-part.html

An example:

```go
package main

import "fmt"

type Book struct {
    pages int
}

func (b Book) SetPages(pages int) {
    b.pages = pages
}

func main() {
    var b Book
    b.SetPages(123)
    fmt.Println(b.pages) // 0
}
```

Another example:

```go
package main

import "fmt"

type Book struct {
    pages int
}

type Books []Book

func (books Books) Modify() {
    // Modifications on the underlying part of the receiver
    // will be reflected to outside of the method.
    books[0].pages = 500
    // Modifications on the direct part of the receiver
    // will not be reflected to outside of the method.
    books = append(books, Book{789})
}

func main() {
    var books = Books{{123}, {456}}
    books.Modify()
    fmt.Println(books) // [{500} {456}]
}
```

Some off topic, if the two lines in the orders of the above `Modify` method are 
exchanged, then both of the modifications will not be reflected to outside of 
the method body.

```go
func (books Books) Modify() {
    books = append(books, Book{789})
    books[0].pages = 500
}

func main() {
    var books = Books{{123}, {456}}
    books.Modify()
    fmt.Println(books) // [{123} {456}]
}
```

The reason here is that the `append` call will allocate a new memory block to 
storage the elements of the copy of the passed slice receiver argument. The 
allocation will not reflect to the passed slice receiver argument itself.

To make both of the modifications be reflected to outside of the method body, 
the receiver of the method must be a pointer one.

```go
func (books *Books) Modify() {
    *books = append(*books, Book{789})
    (*books)[0].pages = 500
}

func main() {
    var books = Books{{123}, {456}}
    books.Modify()
    fmt.Println(books) // [{500} {456} {789}]
}
```

### Should A Method Be Declared With Pointer Receiver Or Value Receiver?

Firstly, from the last section, we know that sometimes we must declare methods 
with pointer receivers.

In fact, we can always declare methods with pointer recievers without any logic 
problems. It is just a matter of program performance that sometimes it is better 
to declare methods with value receivers.


For the cases value receivers and pointer receivers are both acceptable, here 
are some factors needed to be considered to make decisions.

  - Too many pointer copies may cause heavier workload for garbage colllector.
  - If the value size of a receiver type is large, then the receiver argument 
    copy cost may be not neglectable. In particular if the passed argument is an 
    interface value (please read [polymorphism][6] for details), there will be two 
    copies made in the argument passing. Here we just need to know that pointer 
    values are all [small sized values][7]. In fact, for the standard Go compiler 
    and runtime, types other than array and struct types are all small sized types. 
    Struct types with very few fields are also small sized.
  - Mixing value receivers and pointer receivers for the same base type is more 
    likely to cause data races if the declared methods are called concurrently in 
    multiple goroutines.
  - Values of the types in the `sync` standard package should not be copied, so 
    defining methods with value receivers for sturct types which embedding the 
    types in the `sync` standard package is problematic.

If it is hard to make a decision whether a method should use a pointer receiver 
or a value receiver, then just choose the pointer receiver way.

[Github][8] and [Gitlab][9]. Welcome to improve ***Go 101*** articles by 
submitting corrections for all kinds of mistakes, such as typos, grammar errors, 
wording inaccuracies, description flaws, code bugs and broken links.

Support Go 101 by playing [Tapir's games][10].

[6]: interface.html#polymorphism
[7]: value-copy-cost.html
[8]: https://github.com/go101/go101
[9]: https://gitlab.com/Go101/go101
[10]: https://www.tapirgames.com

******************

#### Note (by jfxue):

<a id="note1">
1. 首先，对于 Go 代码显式定义了值接收器的方法，编译器确实会自动生成一个带有指针
   接收器的方法，它其实是通过调用值接收器方法实现的，需要额外做一些参数、返回值
   转换的事情。但是对于文中提到的会自动生成两个隐式的函数不太认同，通过输出汇编
   代码清单，可以看到有两个方法/函数(`"".Book.Pages` 和 `"".(*Book).Pages`)，那么不
   不管是调用值接收器方法，还是指针接收器方法或者直接以函数方式调用，最终都是调
   用这两个 entry，那么从底层来说 method 完全可以看成是第一个参数为接收器的函数。
   只不过 Go 允许我们显式地用调用函数（第一个参数传接收器）的方式来使用罢了。
</a>
