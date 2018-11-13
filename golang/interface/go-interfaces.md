# [Go Interfaces][1]

November 27, 2009

[1]: https://www.airs.com/blog/archives/277


One of the interesting aspects of the Go language is interface objects.  In Go, 
the word interface is overloaded to mean several different things.

Every type has an interface, which is the set of methods defined for that type. 
This bit of code defines a struct type `S` with one field, and defines two 
methods for `S`.

```go
type S struct { i int } 

func (p *S) Get() int { 
    return p.i 
} 

func (p *S) Put(v int) { 
    p.i = v 
}
```

You can also define an interface type, which is simply a set of methods.  This 
defines an interface `I` with two methods:

```go
type I interface {   
    Get() int;   
    Put(int); 
}
```

`S` is a valid implementation for `I`, because it defines the two methods which 
`I` requires. Note that this is true even though there is no explicit 
declaration that `S` implements `I`. A Go program can use this fact via yet 
another meaning of interface, which is an interface value:

```go
func f(p I) { 
    fmt.Println(p.Get()); p.Put(1) 
}
```

Here the variable `p` holds a value of interface type. Because `S` implements 
`I`, we can call `f` passing in a pointer to a value of type `S`:

```go
var s S; f(&s)
```

The reason we need to take the address of `S`, rather than a value of type `S`, 
is because we defined the methods on `S` to operate on pointers. This is not a 
requirement—we could have defined the methods to take values—but then the `Put` 
method would not work as expected.

The fact that you do not need to declare whether a type implements an interface 
means that Go implements a form of duck typing. This is not pure duck typing, 
because when possible the Go compiler will statically check whether the type 
implements the interface. However, Go does have a purely dynamic aspect, in that 
you can convert from one interface type to another. In the general case, that 
conversion is checked at runtime.  If the conversion is invalid—if the type of 
the value stored in the existing interface value does not satisfy the interface 
to which it is being converted—the program will fail with a runtime error.

For example, since every type satisfies the empty interface `interface {}`:

```go
func g(i interface{}) int { 
    return i.(I).Get() 
} 

func h() {   
    var s S;
    fmt.Println(g(&s));   
    fmt.Println(g(s)); // will fail at runtime 
}
```

The first call to `g` will work fine and will print `0`. The second call will 
fail at runtime; when using gccgo, the program will print

`panic: interface conversion failed: no 'Get' method`

This is because, as discussed above, a value of type `S` rather than `*S` does 
not have any methods.

So, how does this work? I will describe the current gccgo implementation. The 
implementation used in the 6g/8g compiler is generally similar but is different 
in important respects.

For every type which is converted to an interface type, gccgo will build a type 
descriptor. Type descriptors are used by the type reflection support in the 
`reflect` package, and they are also used by the internal runtime support. Type 
descriptors are defined in the file `libgo/runtime/go-type.h` in the source 
code. The relevant part here is that for each type they define a (possibly 
empty) list of methods. For each method five pieces of information are stored: a 
hash code for the type of the method (i.e., the parameter and result types); the 
name of the method; for a method which is not exported, the name of the package 
in which it is defined; the type descriptor for the type of the method; a 
pointer to the function which implements the method.

In gccgo, an interface value is really a struct with three fields (
`libgo/runtime/interface.h`): a pointer to the type descriptor for the type of 
the value currently stored in the interface value; a pointer to a table of 
functions implementing the methods for the current value; a pointer to the 
current value itself.

The table of functions is stored sorted by the name of the method, although the 
name does not appear in the table. Calling a method on an interface means 
calling a specific entry in this table, where the entry to call is known at 
compile time. Thus the table of functions is essentially the same as a C++ 
virtual function table, and calling a method on an interface requires the same 
series of steps as calling a C++ virtual function: load the address of the 
virtual table, load a specific entry in the table, and call it.

When a value is statically converted to an interface type, the gccgo compiler 
will build the table of methods required for that value and that interface type. 
This table is specific to the pair of types involved. gccgo will give this table 
comdat linkage, so that it is only built once for each pair of types in a 
program. Thus a static conversion to an interface simply requires loading the 
three fields of the interface struct with values known at compile time.

A dynamic conversion from one interface type to another is more complex.  Of the 
three fields in an interface value, the type descriptor and the pointer to the 
real value can simply be copied to the new interface value. However, the table 
of methods must be built at runtime. This is done by looking at the list of 
methods defined in the value’s type descriptors and the list of methods defined 
in the type descriptor for the interface type itself. Both lists are sorted by 
the name of the method. The runtime code (`libgo/runtime/go-convert-interface.c`) 
merges the two sorted lists to produce the method table. The merging is done 
using the name of the method and the type hash code. If the interface requires a 
method which the type does not provide, the conversion fails.

Interfaces in Go are similar to ideas in several other programming languages: 
pure abstract virtual base classes in C++; typeclasses in Haskell; duck typing 
in Python; etc. That said, I’m not aware of any other language which combines 
interface values, static type checking, dynamic runtime conversion, and no 
requirement for explicitly declaring that a type satisfies an interface. The 
result in Go is powerful, flexible, efficient, and easy to write.

