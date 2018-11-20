[Pointing to Go — The Go Pointer Type][1]

[1]: 
[2]: https://medium.com/learning-the-go-porgramming-language/types-in-the-go-programming-language-65e945d0a692


In my introductory write up on Go types, I did a (rather length) summarized 
walk-through of the basic and composite types in Go (read it [**here**][2] if 
you need a refresher). Continuing with this theme, this write up discusses the 
pointer type: how to create and initialize pointer types.

Though these are easy concepts in Go, nevertheless, a well-grounded 
understanding of these topics is certain to lessen frustrating moments if you 
are a newcomer to Go!

### The Pointer Type

The pointer type in Go is used to point to a memory address where data is 
stored. Similar to C/C++, Go uses the **`*`** operator to designate a type as a 
pointer. The following snippet shows several pointers with different underlying 
types.

```go
var valPtr *float32
var countPtr *int
```

```go
type person struct{name string, age int}
var prsn *person
var matrix *[1024]int
var row []*int64
```

The code snippet above declares each variable as a pointer to its associated 
type:

  - variables *valPtr* and *countPtr* are pointers to their respective primitive 
    types *float32* and *int*
  - variables *prsn* and *matrix* are declared as pointers to their respective 
    composite types *person* and *`[1024]int`*.
  - Lastly, variable *row* is a slice of pointers to elements to type *int64*.

Each pointer stores an address which points to a memory location where a value, 
of the indicated underlying type, is stored. Go does not allow pointer 
arithmetic and, as you have come to expect with Go’s strict type system, each 
pointer type is unique. Meaning, the following will not compile.

```go
var intPtr *int
var int32Ptr *int32
```

```go
intPtr = int32Ptr
```

```
$> cannot use int32Ptr (type *int32) as type *int in assignment
```

This is because a pointer to an *int* is not compatible with pointer of type 
*int32*, even when both point to types with similar memory layout.

#### The Address Operator

Go uses the ***&*** (ampersand) operator to return the address of a variable. 
For instance, the following uses the address operator in expressions that return 
memory locations for associated values.

```go
val := float32(5.5)
var valPtr *float32 = &val
```

```go
score := 79
scorePtr := &score
```

```go
func printId(id *string) { 
    ... 
}
uid := "abcd-eff-33cc-5534"
printId(&uid)
```

The two variables *valPtr* and *scorePtr*, in the pervious code snippet, are 
assigned memory addresses with expressions *`&val`* and *`&score`* respectively. The 
second portion of the code snippet shows function *`printId(id *string)`* that 
takes a pointer as its sole parameter. It is invoked with the address value 
*`&uid`* as its argument.

It is important to note that in Go, you cannot use **`&`** operator directly 
on literal constants for numeric, string, boolean types. For instance, the 
following will not compile.

```go
ptr := &5
```

```
$> cannot take the address of 5
```

You can, however, apply the **`&`** operator to composite type literal 
expressions. For instance, the following shows the literal expressions for 
initializing a struct and an array values that use the ampersand operators to 
return their addresses.

```go
type person struct{name string, age int}
prsn := &person{"Prince", 57}
```

```go
pair := &[2]string{"left-sock", "right-sock"}
```

In the previous snippet, variable *prsn* stores the address of a value of 
*`&person{“Prince”, 57}`*. Variable *pair* is declared and initialized with the 
address of value *`&[2]string{“left-sock”, “right-sock”}`*.

#### Pointer Indirection — Dereference Pointer Values

As established, a pointer is simply an address of an actual value in memory. To 
access that value at the end of the pointer, simply apply the `*` operator to an 
address as shown in the following.

```go
func main() {
    a := 71
    print(&a)
}

func print(val *int) {
    fmt.Println(val)
    fmt.Println(*val * 2)
}
```

In the previous example, *`function print(val *int)`* accepts a pointer as its 
parameter. It first prints the address with the first *`fmt.Println(val)`* . In 
the second *`fmt.Println(*val * 2)`* statement, the code uses pointer 
indirection *`*val`* to access the actual value pointed to, which is then 
multiplied by 2. When executed, the output will look something like the 
following.

```
0x10434114
142
```

#### The new() Function

When the built-in function `new(<type>)` is used to initialize a value, it 
allocates the appropriate memory for a zero-value of the specified type. The 
function then returns the address for the newly created zero-value.

```go
intPtr := new(int)
*intPtr = 77
```

```go
type person struct{name string, age int}
prsn := new(person)
prsn.first = "Prince"
prsn.age = 57
```

The previous snippet uses the *`new()`* function to initialize a zero-value int 
and assign its address to variable *intPtr*. Then the code uses pointer 
indirection to assign it a value with *`*intPtr = 77`*.

Similarly, the code initializes variable *prsn* with the address of zero-value 
for composite type *person*. To access the value (not the address) of the 
pointed composite, the idiom is more forgiving. It is not necessary to write 
*`*prsn.first = “Prince”`*. We can drop the indirection and just use 
*`prsn.first = “Prince”`*.

#### Conclusion

In this short(er) write up, I explored the Go pointer type. You saw how to 
create and initialize pointers. We also explored pointer indirection to access 
and update pointed values. Future write ups will continue to explore the Go type 
system covering Functions, Methods, and Interfaces, etc. (Making the write up 
shorter will hopefully let me get to them sooner!)

Happy go(ding)!

