
[Go Data Structures][1]
=======================

by [Russ Cox](https://swtch.com/~rsc/)

November 24, 2009.

[1]: https://research.swtch.com/godata

When explaining Go to new programmers, I've found that it often helps to explain 
what Go values look like in memory, to build the right intuition about which 
operations are expensive and which are not.  This post is about basic types, 
structs, arrays, and slices.

Basic types
-----------

Let's start with some simple examples:

![](http://research.swtch.com/godata1.png)

The variable `i` has type `int`, represented in memory as a single32-bit word. 
(All these pictures show a 32-bit memory layout; in the current implementations, 
only the pointer gets bigger on a 64-bit machine—`int` is still 32 bits—though 
an implementation could choose to use 64 bits instead.)

The variable `j` has type `int32`, because of the explicit conversion. Even 
though `i` and `j` have the same memory layout, they have different types: the 
assignment `i = j` is a type error and must be written with an explicit 
conversion: `i = int(j)`.

The variable `f` has type `float`, which the current implementations represent 
as a 32-bit floating-point value. It has the same memory footprint as the 
`int32` but a different internal layout.

Structs and pointers
--------------------

Now things start to pick up. The variable `bytes` has type `[5]byte`, an array 
of 5 `byte`s. Its memory representation is just those 5 bytes, one after the 
other, like a C array. Similarly, `primes` is an array of 4`int`s.

Go, like C but unlike Java, gives the programmer control over what is and is not 
a pointer. For example, this type definition:

```go
type Point struct { X, Y int }
```

defines a simple struct type named `Point`, represented as two adjacent`int`s in 
memory.

![](http://research.swtch.com/godata1a.png)

The [composite literal syntax][2]`Point{10, 20}` denotes an initialized `Point`. 
Taking the address of a composite literal denotes a pointer to a freshly 
allocated and initialized `Point`. The former is two words in memory; the latter 
is a pointer to two words in memory.

[2]: http://golang.org/doc/go_spec.html#Composite_literals

Fields in a struct are laid out side by side in memory.

```go
type Rect1 struct { Min, Max Point }
type Rect2 struct { Min, Max *Point }
```

![](http://research.swtch.com/godata1b.png)

`Rect1`, a struct with two `Point` fields, is represented by two`Point`s—four 
ints—in a row. `Rect2`, a struct with two `*Point` fields, is represented by two 
`*Point`s.

Programmers who have used C probably won't be surprised by the distinction 
between `Point` fields and `*Point` fields, while programmers who have only used 
Java or Python (or ...) may be surprised by having to make the decision. By 
giving the programmer control over basic memory layout, Go provides the ability 
to control the total size of a given collection of data structures, the number 
of allocations, and the memory access patterns, all of which are important for 
building systems that perform well.

Strings
-------

With those preliminaries, we can move on to more interesting data types.

![](http://research.swtch.com/godata2.png)

(The gray arrows denote pointers that are present in the implementation but not 
directly visible in programs.)

A `string` is represented in memory as a 2-word structure containing a pointer 
to the string data and a length. Because the `string` is immutable, it is safe 
for multiple strings to share the same storage, so[slicing][3]`s` results in a 
new 2-word structure with a potentially different pointer and length that still 
refers to the same byte sequence. This means that slicing can be done without 
allocation or copying, making string slices as efficient as passing around 
explicit indexes.

(As an aside, there is a [well-known gotcha][4] in Java and other languages that 
when you slice a string to save a small piece, the reference to the original 
keeps the entire original string in memory even though only a small amount is 
still needed. Go has this gotcha too. The alternative, which we tried [and 
rejected][5], is to make string slicing so expensive—an allocation and a copy—
that most programs avoid it.)

[3]: http://www.blogger.com/post-edit.g?blogID=8082954141980125536&postID=65253524121904390
[4]: http://bugs.sun.com/bugdatabase/view_bug.do?bug_id=4513622
[5]: http://code.google.com/p/go/source/detail?r=70fa38e5a5bb
[6]: http://golang.org/doc/effective_go.html#slices
[7]: http://golang.org/doc/effective_go.html#slices

Slices
------

![](http://research.swtch.com/godata3.png)

A [slice][6] is a reference to a section of an array. In memory, it is a 3-word 
structure contaning a pointer to the first element, the length of the slice, and 
the capacity. The length is the upper bound for indexing operations like`x[i]` 
while the capacity is the upper bound for slice operations like`x[i:j]`.

Like slicing a string, slicing an array does not make a copy: it only creates a 
new structure holding a different pointer, length, and capacity. In the example, 
evaluating the composite literal`[]int{2, 3, 5, 7, 11}` creates a new array 
containing the five values and then sets the fields of the slice `x` to describe 
that array. The slice expression `x[1:3]` does not allocate more data: it just 
writes the fields of a new slice structure to refer to the same backing store. 
In the example, the length is 2—`y[0]` and `y[1]` are the only valid indexes—but 
the capacity is 4—`y[0:4]` is a valid slice expression. (See[Effective Go][7] 
for more about length and capacity and how slices are used.)

Because slices are multiword structures, not pointers, the slicing operation 
does not need to allocate memory, not even for the slice header, which can 
usually be kept on the stack. This representation makes slices about as cheap to 
use as passing around explicit pointer and length pairs in C. Go originally 
represented a slice as a pointer to the structure shown above, but doing so 
meant that every slice operation allocated a new memory object. Even with a fast 
allocator, that creates a lot of unnecessary work for the garbage collector, and 
we found that, as was the case with strings above, programs avoided slicing 
operations in favor of passing explicit indices. Removing the indirection and 
the allocation made slices cheap enough to avoid passing explicit indices in 
most cases.

New and Make
------------

Go has two data structure creation functions: `new` and `make`. The distinction 
is a common early point of confusion but seems to quickly become natural. The 
basic distinction is that `new(T)` returns a `*T`, a pointer that Go programs 
can dereference implicitly (the black pointers in the diagrams), while `make(T, 
`*args*`)` returns an ordinary `T`, not a pointer. Often that `T` has inside it 
some implicit pointers (the gray pointers in the diagrams). `New` returns a 
pointer to zeroed memory, while `make` returns a complex structure.

![](http://research.swtch.com/godata4.png)

There is a way to unify these two, but it would be a significant break from the 
C and C++ tradition: define `make(*T)` to return a pointer to a newly allocated 
`T`, so that the current `new(Point)` would be written`make(*Point)`. We tried 
this for a few days but decided it was too different from what people expected 
of an allocation function.

Coming soon...
--------------

This has already gotten a bit long. Interface values, maps, and channels will 
have to wait for future posts.
