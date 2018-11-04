### [Slices][1]

[1]: https://golang.org/doc/effective_go.html#slices    

Slices wrap arrays to give a more general, powerful, and convenient interface to
sequences of data. Except for items with explicit dimension such as
transformation matrices, most array programming in Go is done with slices rather
than simple arrays.

Slices hold references to an underlying array, and if you assign one slice to
another, both refer to the same array. If a function takes a slice argument,
changes it makes to the elements of the slice will be visible to the caller,
analogous to passing a pointer to the underlying array. A `Read` function can
therefore accept a slice argument rather than a pointer and a count; the length
within the slice sets an upper limit of how much data to read. Here is the
signature of the `Read` method of the `File` type in package `os`:

```go
func (f *File) Read(buf []byte) (n int, err error)
```

The method returns the number of bytes read and an error value, if any. To read
into the first 32 bytes of a larger buffer `buf`, *slice* (here used as a verb)
the buffer.

```go
n, err := f.Read(buf[0:32])
```

Such slicing is common and efficient. In fact, leaving efficiency aside for the
moment, the following snippet would also read the first 32 bytes of the buffer.

```go
var n int
var err error
for i := 0; i < 32; i++ {
    nbytes, e := f.Read(buf[i:i+1])  // Read one byte.
    if nbytes == 0 || e != nil {
        err = e
        break
    }
    n += nbytes
}
```

The length of a slice may be changed as long as it still fits within the limits
of the underlying array; just assign it to a slice of itself. The*capacity* of a
slice, accessible by the built-in function `cap`, reports the maximum length the
slice may assume. Here is a function to append data to a slice. If the data
exceeds the capacity, the slice is reallocated. The resulting slice is returned.
The function uses the fact that `len` and `cap` are legal when applied to the
`nil` slice, and return 0.

```go
func Append(slice, data []byte) []byte {
    l := len(slice)
    if l + len(data) > cap(slice) {  // reallocate
        // Allocate double what's needed, for future growth.
        newSlice := make([]byte, (l+len(data))*2)
        // The copy function is predeclared and works for any slice type.
        copy(newSlice, slice)
        slice = newSlice
    }
    slice = slice[0:l+len(data)]
    copy(slice[l:], data)
    return slice
}
```

We must return the slice afterwards because, although `Append` can modify the
elements of `slice`, the slice itself (the run-time data structure holding the
pointer, length, and capacity) is passed by value.

The idea of appending to a slice is so useful it's captured by the`append`
built-in function. To understand that function's design, though, we need a
little more information, so we'll return to it later.

### Two-dimensional slices

Go's arrays and slices are one-dimensional. To create the equivalent of a 2D
array or slice, it is necessary to define an array-of-arrays or slice-of-slices,
like this:

```go
type Transform [3][3]float64  // A 3x3 array, really an array of arrays.
type LinesOfText [][]byte     // A slice of byte slices.
```

Because slices are variable-length, it is possible to have each inner slice be a
different length. That can be a common situation, as in our`LinesOfText`
example: each line has an independent length.

```go
text := LinesOfText{
    []byte("Now is the time"),
    []byte("for all good gophers"),
    []byte("to bring some fun to the party."),
}
```

Sometimes it's necessary to allocate a 2D slice, a situation that can arise when
processing scan lines of pixels, for instance. There are two ways to achieve
this. One is to allocate each slice independently; the other is to allocate a
single array and point the individual slices into it. Which to use depends on
your application. If the slices might grow or shrink, they should be allocated
independently to avoid overwriting the next line; if not, it can be more
efficient to construct the object with a single allocation. For reference, here
are sketches of the two methods. First, a line at a time:

```go
// Allocate the top-level slice.
picture := make([][]uint8, YSize) // One row per unit of y.
// Loop over the rows, allocating the slice for each row.
for i := range picture {
    picture[i] = make([]uint8, XSize)
}
```

And now as one allocation, sliced into lines:

```go
// Allocate the top-level slice, the same as before.
picture := make([][]uint8, YSize) // One row per unit of y.
// Allocate one large slice to hold all the pixels.
pixels := make([]uint8, XSize*YSize) // Has type []uint8 even though picture is [][]uint8.
// Loop over the rows, slicing each row from the front of the remaining pixels slice.
for i := range picture {
    picture[i], pixels = pixels[:XSize], pixels[XSize:]
}
```

Except as[noted][8], the content of this page is licensed under the Creative 
Commons Attribution3.0 License,  and code is licensed under a [BSD license][9]
[Terms of Service][10] \| [Privacy Policy][11]

[8]: https://developers.google.com/site-policies#restrictions
[9]: https://golang.org/LICENSE
[10]: https://golang.org/doc/tos.html
[11]: https://www.google.com/intl/en/policies/privacy/
