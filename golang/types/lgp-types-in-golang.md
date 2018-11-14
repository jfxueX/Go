# [Types in the Go Programming Language][1]

by Vladimir Vivien

[Learning the Go Programming Language][2]

[1]: https://medium.com/learning-the-go-programming-language/types-in-the-go-programming-language
[2]: https://medium.com/learning-the-go-programming-language


* [Strictly Typed](#strictly-typed)
    * [Variable Declaration & Initialization](#variable-declaration--initialization)
    * [Zero Value](#zero-value)
* [Pre-Declared (Named) Types](#pre-declared-namedtypes)
    * [Strings](#strings)
    * [**Integers**](#integers)
    * [Floats](#floats)
    * [Boolean](#boolean)
* [Composite (Unnamed) Types](#composite-unnamed-types)
    * [Arrays](#arrays)
    * [Slices](#slices)
    * [Map](#map)
    * [Structs](#structs)
* [Type Declaration](#type-declaration)
* [Type Conversion](#type-conversion)
* [Conclusion](#conclusion)

One aspect of the Go language that may cause newcomers to scratch their heads in 
wonder is the type system. Go types are simple and straightforward. However, a 
grounded understanding of the fundametal characteristics of Go types helps 
lowering the number of frustrating moments when adopting the language. This 
writeup presents a condensed (really, it is) summary of types and type system 
rules that new Go programmers and even current practitioners may find useful.

> This writeup leaves out discussions on pointer, function, and interface types
> to keep things short(er). These topics may be discussed in future writeups.

### Strictly Typed

As a strictly typed language, Go has a rich type system with a multitude of 
types to represent data of many forms. Unlike a language like Java (where there 
are only primitive and reference types), Go has types to represent textual, 
numeric, boolean, pointer, composite, function, and interface values.

As we will cover in detail later, each type is distinct. Once a variable is 
declared to be of a certain type, it can only carry values of that type. This is 
true even with types of similar memory layout. For instance, the following will 
break at compilation without an explicit conversion.

```go
var counter int
counter = 5
var mask int32
mask = counter    // type mismatch
```

#### Variable Declaration & Initialization

Let’s quickly review how variables are declared and initialized. If you know 
this stuff already, skip ahead to the next section. As you saw above, variables 
can be declared with the key word *var* followed by an identifier and the type 
of the variable. A variable can also be initialized in the declaration step as 
shown below.

```go
var counter int = 5
var message string = "Hello"
```

This is known as the long form declaration. We can actually drop the type on the 
right side of the variable identifier and let the type system automatically 
assign type *int* to variable *counter* and type *string* to *message*.

```go
var counter = 5
var message = "Hello"
```

There’s even a shorter form (only allowed inside a function) that is commonly 
used in Go code where the **var** keyword is dropped. It declares a variable and 
assigns it a value using the “:=” operator as shown below.

```go
func main(){
    counter := 5
    message := "Hello"
}
```

Note that when a variable is declared without a specific type (as in the case 
above) the type system will use the assigned value literal to determine the type 
assigned to the variable. From the above example, variable *counter* will be an 
*int* and variable *message* will be a *string* (detailed later).

#### Zero Value

Lastly, one common aspect of all types in Go is the notion of zero value. When a 
variable is declared, and not explicitly initialized, it will be allocated 
storage with a default value, appropriate for the type, known as the type’s 
*zero value*. The following illustrates some zero values.

```go
var message string  // default value ""
var factor float32  // default value 0
var enabled bool    // false
```

### Pre-Declared (Named) Types

Most types in Go can be given an identifier (or name) that represents the type. 
Go, for instance, comes with a set of built-in pre-declared (named) primitive 
types that can be used to represent textual, numeric, and boolean values.

#### Strings

They are immutable text values that are utf-8 encoded to represent unicode 
characters as shown below. The zero value of an uninitialized string is the 
empty string “” (not the nil reference). String values can be initialized with 
text literals surrounded by double quotes. The following shows a string with an 
embedded Unicode value of 00*B0* escaped with *\\u*.

```go
message := "It is 42\u00B0 F outside!"
```

```go
// printed as
It is 42° F outside!
```

String values can also be surrounded by back quotes (grave accents) \`\` for raw 
multi-line unencoded text as shown in the following example.

```go
message := `
It is 42 \u00B0 F 
outside!
`
```

```go
// prints
It is 42 \u00B0 F outside!
```

#### **Integers**

Go has several types, with different internal sizes, that can be used to store 
integer values as listed below:

-  **int{8,16,32,64}**  
   singed integers of 8,16,32,64 bit in size (*int32*, *int64*, etc)
-  **uint{8,16,32,64}**  
   unsigned integers of 8,16,32,64 bit in size (i.e.  *uint8*)
-  **byte**  
   alias for and equivalent to *uint8*
-  **rune**  
   used to represent characters, alias for and equivalent to *int32*
-  **int**  
   signed integers of at least 32-bit in size, **not** equivalent to *int32*
-  **uint**  
   unsigned integers of at least 32-bit; **not** equivalent to *uint32*
-  **uintptr**  
   dedicated for storing memory address pointers

Uninitialized, integral types have a zero value of 0. Integers can be 
initialized with constant literals which can be expressed as a decimal, octal, 
and hex as shown below

```go
var color uint32 = 0xFEFEFE  // hex (0x prefix)
var mod = 0466               // octal (0 prefix)
count := 1245                // decimal
```

When untyped variables (*mod* and *count* for instance) are initialized with 
integer literals, the type system automatically assign type *int* to these 
variables.

#### Floats

Floats can have types float32 and float64 represented with 32 and 64-bit sizes 
in memory respectively. An uninitialized float has a zero value of 0. Floating 
point values can be initialized with decimal literals that can include a decimal 
point and an exponent as shown.

```go
var pi float32 = 3.1415
avogadro := 6.0221409e+1
```

Note that in cases of an untyped variable declaration, the type system will 
automatically assign *float64* to a float variable.

Go can also represent complex numbers with types *complex64* and *complex128*. 
Each type use *float32* and *float64*, respectively, to represent their real and 
imaginary parts.

#### Boolean

Boolean values can be stored in the *bool* type in Go. This type can only be 
assigned pre-declared values of *true* or *false*. The zero value of a boolean 
value is *false*. The following declares variable *enabled* and assign type bool 
to it with initial value of true.

```go
enabled := false
```

### Composite (Unnamed) Types

Go supports the notion of composite types, such as arrays, slices, maps, and 
structs as building blocks for new types. *Composite types are known as “unnamed 
types”*, because *they use a type literal to represent the structural definition 
of the type, instead of using a simple name identifier*.

Unlike its named counterpart, unnamed composite types use literals for value 
initialization that are composed of type (itself) and a literal text that 
represents the value. As you read this section pay attention how literals are 
formed for arrays, slices, maps, and structs.

#### Arrays

Go arrays are containers for storing sequenced values, of the same type, that 
are numerically indexed. For instance, the following declares variable *triple* 
as a 3-element integer array type *`[3]int`*.

```go
var triple [3]int
```

**It is important** to understand that the type for variable *triple* is 
*`[3]int`*. This means *triple* can only be assigned 3-element *int* array 
values. The following will cause a compilation error.

```go
var double [2]int
double = triple
```

```bash
$> cannot use triple (type [3]int) as type [2]int in assignment
```

The zero value of an uninitialized array is pre-filled with the zero value of 
the array’s declared element as shown below.

```go
var double [2]int    // zero value [0, 0]
var steps [3]string  // zero value ["", "", ""]
```

Arrays are initialized with a composite literal value that represents the array 
type and the sequence of values in the array. The following example declares and 
initializes a 3-element array.

```go
var steps [3]string = [3]string{"SEND", "RCVD", "WAIT"}
```

This can be simplified using the shorter form of declaration and initialization 
as follows.

```go
steps := [3]string{"SEND", "RCVD", "WAIT"}
```

Note that arrays are a bit inflexible as a container type. The size of an array 
is not dynamic. In addition, as soon as the size of an array changes, it becomes 
a new type (i.e. *`[3]string`* is different from *`[4]string`*) which makes it 
cumbersome for type reuse.

#### Slices

Given the limitations of arrays above, the slice is the idiomatic type used for 
sequentially ordered types. The slice is a composite type with semantics similar 
to arrays. In fact, a slice uses an array as its underlying datastore.

```go
steps := []string{"SEND", "RCVD", "WAIT"}
```

The obvious difference, in the previous code snippet, is the absence of the size 
in the type specifier which means the slice type can represent all sets of the 
declared element.

The zero value of an uninitialized slice is the reference value of *nil*. The 
following will cause a runtime panic of a Go program.

```go
var points []int  // uninitialized slice
points[0] = 12    // nil slice, causes panic
```

Before a slice is ready to be used, it must be initialized using a composite 
literal value (as shown in the snippet earlier for variable *steps*), or it must 
be made using the built-in function **make()**.

```go
points := make([]int,2)
points[0] = 12
points [1] = 24
```

The *make()* function creates and allocates enough memory for the number of 
specified elements (second param) with each element initialized with its zero 
value.

#### Map

Go maps are composite types for storing elements of the same type indexed by an 
arbitrary hash key value. Similar to previous composite types, the map type 
literal expresses the structure of the map by declaring the type of the map key 
and that of its elements.

```go
var row map[string][]string  // key string, elements []string type
var collision map[[2]int]int // key [2]int, elements int type
```

An uninitialized map (as above) has a nil value where any attempt to place data 
will cause a runtime panic. A map value may be initialized using a composite 
literal that specifies the map type followed by its element values.

```go
data := map[string][]int {
   "men":  []int{32, 55, 12, 55, 42, 53},
   "women":[]int{44, 42, 23, 41, 65, 44},
}
```

Using the composite literal form, each element entry, in the map, is composed of 
a value for the key, followed by a “:”, and the value for the element. Similar 
to the slice, a map variable can also be initialized using the *make()* function 
to initialize the underlying storage to receive data.

```go
cal := make(map[string]int)
cal["Jan"] = 100
cal["Feb"] = 445
cal["Mar"] = 514
```

#### Structs

A *struct* is a composite type that stores zero or more elements indexed by a 
named identifier known as a field. The struct type literal specifies the name of 
each field in the struct as shown the following declarations.

```go
var car struct{make, model string}
var currency struct{name, country string; code int}
var node struct{
   edges []string
   weight int
}
var empty struct{}  // an empty struct type
```

It is important to note that, similar to arrays, the entire struct block is the 
type. A struct is equivalent if it is declared the exact same elements in the 
same order. For instance, the following will not compile.

```go
var car struct{make, model string}
var bike struct{model, make string} = car
```

```bash
$> cannot use car (type struct { make string; model string }) as type struct { model string; make string } in assignment
```

As with previous composite types, initializing a struct can be done with a 
composite literal value made up of the struct type followed a set of field 
values as illustrated below.

```go
var car struct{make,model string} = struct{make,model string}{
    "make":  "Ford",
    "model": "f150",
}
```

```go
empty := struct{}{} // empty struct initialized
```

### Type Declaration

As you have seen, Go type literals take many forms. It would be absurd (and 
exhausting), however, to have to redeclare a composite type (like a struct for 
instance) every time it is needed.

Luckily, Go supports an idiomatic way to declare types by binding an identifier 
to an existing underlying type. The following declares two types: `engSize` with 
underlying type `uint8` and `vehicle` with underlying type `struct{make, model 
string; engSize}`.

```go
type engSize uint8
type vehicle struct {
    make, model string
    engine engSize
}
```

```go
ford := vehicle{"ford", "f150", 8}
toyo := vehicle{make:"toyota", engine:4}
```

Once declared the newly created type can be used wherever its underlying type is 
allowed. It should be noted that a declared type is considered to be different 
from its underlying built-in type.

```go
type engSize uint8
var size uint8 = 6
var fordEng engSize = size
```

```bash
$> cannot use size (type uint8) as type engSize in assignment
```

The compiler considers type *engSize* to be different from its underlying type 
*uint8* (that is strict typing for you).

### Type Conversion

The last topic for this write up, is that of type conversion. As explained 
earlier, each type is considered different. To cross type boundaries, you must 
use type conversion expressions to convert from one compatible type to another. 
The previous example can be fixed by converting variable *size* to *engSize* as 
follows.

```go
type engSize uint8
var size uint8 = 6
var fordEng engSize = engSize(size)  // conversion expression
```

Here is another example to drive the point home. The following will fail 
compilation because the addition expression is mixing the types.

```go
var count int32
var actual int
var test int64 = actual + count
```

```bash
$> invalid operation: actual + count (mismatched types int and int32)
```

A conversion expression must be applied to cross the boundaries of the different 
types to fix the addition.

```go
var test int64 = int64(int32(actual) + count)
```

### Conclusion

This writeup was meant to be a summary to help new comers understand the rules 
of the Go type systems. It covers built-in and composite types.  There are much 
detail left out, but this should provide a good starting point to understanding 
Go and its types. I intentionally left out discussion on pointers, functions, 
and interfaces as they bring their own implications.

Any feedback is welcome. If this was useful, share and recommend to others 
interested in Go!
