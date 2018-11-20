[Breaking the Type System in Golang (aka dynamic types)][1]

[1]: https://medium.com/@utter_babbage/breaking-the-type-system-in-golang-aka-dynamic-types-8b86c35d897b

> *TL;DR It is possible to change type signatures of go structs during
> runtime and create entirely new types in runtime. This story tells you
> how.*

I want to write this starting with a disclaimer: *The go type system exists for 
a reason, and it’s not a good practice to ever mess with it.  I did this purely 
for lulz.*

Now that’s out of the way, I’ll briefly explain what prompted me to create this 
technique. I was trying to find a quick way to change how a struct gets 
marshaled into JSON. I wanted to change the field name of one of the fields that 
was printed.

Here’s an example of the behavior I wanted

```go
package main
```

```go
import (
   "fmt"
   "encoding/json"
)
```

```go
type example struct {
    someStringFieldsName string   `json:"someStringFieldsName"`
    someIntFieldsName int `json:"someIntFieldsName"`
}
```

```go
func main() {
  data, _:= json.MarshalIndent(&example{}, "", " ")
  fmt.Println(data)
}
```

```json
# Output
{
  "someIntFieldsName": 0
  "someStringFieldsName": ""
}
```

```json
# My intended goal during runtime
{
  "someOtherIntFieldsName": 0            # notice the changed key
  "someOtherStringFieldsName": ""        # notice the changed key
}
```

I was working with a deeply nested type that came from a library that I had no 
control over. I was out of the following options:

1.  <span id="2763">Change the name of the field in the struct. I didn’t
    have any control over the struct.</span>
2.  <span id="4187">Write a MarshalJSON function for the type. This
    again boils down to not having control over the type.</span>
3.  <span id="909e">I needed to do this for many types in a scalable
    way. I had to do this for about 500 different type objects. I didn’t
    want to create my own types for each of the types and them make them
    marshal my way.</span>
4.  <span id="7437">I had no control over the encoder this library was
    using, unless I re-architected the entire library in a big
    way.</span>

So, I began by looking at different options. I looked at various blog posts, and 
stack overflow answers, and everyone said one thing — **it can’t be done.** 
*That only made me want to do it more.*

I looked into how the types are represented in memory, and how its associated 
data is used by the compiler. Thanks to go being open source, I was able to find 
this easily by reading the compiler’s code.

For eg. A string is represented in memory this way

|0000 0000|0000 0000|0000 0000| 0000 0000| 0000 0000| 0000 0000| 0000
0000| 0000 0000| …. (another 8 bytes to represent the pointer to the
actual data)

The above sequence of 8 bytes (or whatever the size of int is on your platform) 
represents the length of the string. The following 8 bytes(or the size of 
\`uintptr\` on your platform) is a pointer to the actual slice data. So, if the 
pointer points to a byte array of length in the first 8 bytes, then the pointer 
to the first 8 bytes can be “unsafely”“typecast” into a string.

This memory layout can be represented using a struct

``` graf graf--pre graf-after--p
type stringHeader struct { 
  data unsafe.Pointer
  len int
}
```

now, using the unsafe package, it is possible to convert any string into the 
above struct and manipulate that struct. For eg.

``` graf graf--pre graf-after--p
func convertSliceToStruct() {
  s := "hello"
  t := "world"
  sPrime := (*stringHeader)(unsafe.Pointer(&s))
  tPrime := (*stringHeader)(unsafe.Pointer(&t))
```

```go
  sPrime.len = 4
  fmt.Println(s) // this will print hell instead of hello
```

```go
  sPrime.data = tPrime.data // same as s = t, except length is 4
  fmt.Println(s) // this will print worl
} 
```

Go has a more interesting layout for structs. The best way to understand this is 
by converting a struct to an interface{}. When you do this, the interface data 
layout looks like this.

``` graf graf--pre graf-after--p
type interfaceHeader struct{
  typ uintptr
  word unsafe.Pointer
}  // 8 bytes for the pointer to the actual struct data followed by
   // 8 bytes for the pointer to the type information
```

now, using the unsafe package, it is possible to convert any struct wrapped in 
an interface to this type. The typ field points to a struct of the type 
structType. That structure looks like this

``` graf graf--pre graf-after--p
// structType represents a struct type.
type structType struct { 
  rtype   
  pkgPath name
  fields  []structField // sorted by offset
}
```

where name, structField and rtype look like

``` graf graf--pre graf-after--p
type name struct { 
 bytes *byte
}
```

```go
// Struct field
type structField struct { 
 name       name    // name is always non-empty
 typ        *rtype  // type of field
 offsetAnon uintptr // byte offset of field<<1 | isAnonymous
}
```

```go
type rtype struct { 
 size       uintptr 
 ptrdata    uintptr  // number of bytes that can contain pointers
 hash       uint32   // hash of type; 
 tflag      tflag    // extra type information flags
 align      uint8    // alignment of variable with this type
 fieldAlign uint8    // alignment of struct field with this type 
 kind       uint8    // enumeration for C
 alg        *typeAlg // algorithm table
 gcdata     *byte    // garbage collection data
 str        nameOff  // string form
 ptrToThis  typeOff  // type for pointer to this type, may be zero
}
```

The name structure is an interesting one. It contains both name of the field and 
the tag of the field.

The first byte of the name struct holds metadata about the field.

``` graf graf--pre graf-after--p
the last bit of the first byte signifies if the field is exported
the second last bit of the first byte denotes if a tag exists
```

```go
the next two bytes denote the length of the name field
the following n bytes are the name of the field
```

```go
if tag exists, the next two fields are the length of the tag
the following m bytes are the value of the tag
```

Once you have these layouts, it is possible to extract the type information such 
as fieldname, fieldtag from any go struct. Let’s take an example struct to 
understand how to do this.

```go
type example struct {
  A int `json:"a,omitempty"`
}
```

```go
func mess() {
 a := example{A: 1}
 var aInterface interface{}
```

```go
 aInterface = a
  
 aHeader := (*interfaceHeader)(unsafe.Pointer(&aInterface))
  
 aTypeStruct := (*structType)(unsafe.Pointer(aHeader.typ))
  
 // possible to iterate through the fields 
 for i := range aTypeStruct.fields {
   f := aTypeStruct.fields[i]
```

```go
   //extract interesting bytes
   meta:= (*[4]bytes)(unsafe.Pointer(f.name.bytes)) 
```

```go
   //extract name of the struct field 
   nameHeader := &stringHeader{}
   nameHeader.len = int(int(meta[1])<<8 | int(meta[2]))
   nameHeader.data = unsafe.Pointer(&meta[3])
```

```go
   var name string
   name = *(*string)(unsafe.Pointer(nameHeader))
```

```go
   var tag string
```

```go
   //check if tag exists
   if meta[0] & (1<<1) {
      //extract tag - first find the beginning of tag
      // 1 byte for metadata + 2 bytes for len of name
      // + n bytes for name
      tagStart = unsafe.Pointer(unsafe.Pointer(f.name.bytes) + 3 + len(name))
      //since we know tag exists, extract 3 bytes
      tagMeta := (*[3]byte)(tagStart)
      tagHeader := &stringHeader{}
      tagHeader.len = int(int(tagMeta[0])<<8 | int(tagMeta[1]))
      tagHeader.data = unsafe.Pointer(&tagMeta[2])
```

```go
      tag = *(*string)(unsafe.Pointer(tagHeader))
   }  
 
   fmt.Printf("%s %s\n", name, tag)
  }
}
```

```go
# This will output
```

```go
A json:"a,omitempty"
```

now that we know how to extract the types, it should be simple enough to change 
the value right? Let’s try it

``` graf graf--pre graf-after--p
func mess() {
 a := example{}
 var aInterface interface{}
```

```go
 aInterface = a
  
 aHeader := (*interfaceHeader)(unsafe.Pointer(&aInterface))
  
 aTypeStruct := (*structType)(unsafe.Pointer(aHeader.typ))
  
 // possible to iterate through the fields 
 for i := range aTypeStruct.fields {
   f := aTypeStruct.fields[i]
```

```go
   //extract interesting bytes
   meta:= (*[4]bytes)(unsafe.Pointer(f.name.bytes))
```

```go
   //extract name of the struct field 
   nameHeader := &stringHeader{}
   nameHeader.len = int(int(meta[1])<<8 | int(meta[2]))
   nameHeader.data = unsafe.Pointer(&meta[3])
```

```go
   var name string
   name = *(*string)(unsafe.Pointer(nameHeader))
```

```go
   //let's try to set a new name
   newName := "b"
   newNameHeader = (*stringHeader)(unsafe.Pointer())
```

```go
   // 1 byte for metadata + 2 bytes for length + 1 byte for "b"
   newNameBytes := &[4]byte{}
   newNameBytes[0] = byte(0x01) // exported field
   newNameBytes[1] = byte(uint8(len(newName) >> 8))
   newNameBytes[2] = byte(uint8(len(newName)))
   newNameBytes[3] = *(*byte)(newNameHeader.data)
```

```go
//set this new value into the field
   f.name.bytes = (*byte)(unsafe.Pointer(&newNameBytes))
   fmt.Printf("%s %s\n", name, tag)
  }
}
```

So, let’s run this.

``` graf graf--pre graf-after--p
unexpected fault address 0x4a3a11
fatal error: fault
[signal SIGSEGV: segmentation violation code=0x2 addr=0x4a3a11 pc=0x49a7ad]
```

```go
goroutine 1 [running]:
runtime.throw(0x4ce6f4, 0x5)
        /usr/local/go/src/runtime/panic.go:596 +0x95 fp=0xc42004dca0 sp=0xc42004dc80
runtime.sigpanic()
        /usr/local/go/src/runtime/signal_unix.go:297 +0x28c fp=0xc42004dcf0 sp=0xc42004dca0
```

Oops, a runtime panic\!

Hmmm, this should’ve worked right? We did the same kind of changes for
the strings above and that seemed to work. Well, let’s debug why this is
not working. First, let’s examine the addresses of the struct object,
and also the address of the type struct that we are trying to
manipulate.

``` graf graf--pre graf-after--p
func mess() {
 a := example{}
 var aInterface interface{}
```

```go
 aInterface = a
  
 aHeader := (*interfaceHeader)(unsafe.Pointer(&aInterface))
  
 aTypeStruct := (*structType)(unsafe.Pointer(aHeader.typ))
```

```go
 fmt.Printf("obj=%p obj_type=%p\n", &a, aTypeStruct)
}
```

```go
# The output will look like
obj=0xc420010370 obj_type=0x4b98e
```

If you notice, the obj pointer looks longer than the address to the type
pointer. The pointer to the object type data is only 24 bits long. The
object pointer however is 40 bits long. So, I began to objdump the
compiled binary.

That’s when I saw the value of the addresses in the objdump files. The
addresses there also were 24 bits long. That’s what led me to realize
that the type information is stored in the code segment of the program.
The code segment is memory protected by default and contains the
executable instructions of the program itself.

The `segment violation error` earlier happened because the program was
writing to a protected segment. So, I went ahead and did the right thing
and stopped there.

***Nope***

I still wanted to change the type information. So, I went ahead and made
that section of the memory writable. This can be done using the
`mprotect` syscall. This syscall works at the page level (mem-aligned)
and can make entire pages readable, writable or executable (or a
combination of the three). I issued a `mprotect` syscall to the page in
which the name information of the type resided. That made that whole
page readable, writable and executable. Then, the pointer assignment
above worked.

That’s all\! Here’s the output I got.

``` graf graf--pre graf-after--p
package main
```

```go
import (
  "fmt"
  "encoding/json"
  "unsafe"
)
```

```go
type example struct {
 A int 
}
```

```go
...  //type definitions for all the layout types
```

```go
func mess() example {
 a := example{A: 1}
 var aInterface interface{}
```

```go
 aInterface = a
  
 aHeader := (*interfaceHeader)(unsafe.Pointer(&aInterface))
  
 aTypeStruct := (*structType)(unsafe.Pointer(aHeader.typ))
  
 // possible to iterate through the fields 
 for i := range aTypeStruct.fields {
   f := aTypeStruct.fields[i]
```

```go
   //extract interesting bytes
   meta:= (*[4]bytes)(unsafe.Pointer(f.name.bytes))
```

```go
   //extract name of the struct field 
   nameHeader := &stringHeader{}
   nameHeader.len = int(int(meta[1])<<8 | int(meta[2]))
   nameHeader.data = unsafe.Pointer(&meta[3])
```

```go
   var name string
   name = *(*string)(unsafe.Pointer(nameHeader))
```

```go
   //let's try to set a new name
   newName := "b"
   newNameHeader = (*stringHeader)(unsafe.Pointer())
```

```go
   // 1 byte for metadata + 2 bytes for length + 1 byte for "b"
   newNameBytes := &[4]byte{}
   newNameBytes[0] = byte(0x01) // exported field
   newNameBytes[1] = byte(uint8(len(newName) >> 8))
   newNameBytes[2] = byte(uint8(len(newName)))
   newNameBytes[3] = *(*byte)(newNameHeader.data)
```

```go
   // syscall.Mprotect() to make the page writable
```

```go
   //set this new value into the field
   f.name.bytes = (*byte)(unsafe.Pointer(&newNameBytes))
 
   // syscall.Mprotect() to make the page read only again
}
 return a
}
```

```go
func main() {
  preMess := example{A: 1}
  data, err := json.MarshalIndent(&preMess, "", " ")
  if err != nil {
    fmt.Println(err)
    return
  }
  fmt.Printf("%s\n", string(data))
```

```go
  fmt.Println("changing type signature")
```

```go
  val := mess()
  data, err = json.MarshalIndent(&val, "", " ")
  if err != nil {
    fmt.Println(err)
    return
  }
  fmt.Printf("%s\n", string(data))
}
```

```go
# Output
```

```json
{ 
  "A": 1    # Notice the key before changing type signature 
}
changing type signature
{ 
  "b": 1    # Notice the changed key 
}
```

*Notes:*

  - <span id="59dc">I’ve left out some critical details for getting this
    right</span>
  - <span id="9b73">For eg. how to use the mprotect syscall is omitted.
    Hopefully, this will ensure that only those people who truly know
    what they are doing will be able to use this technique</span>
  - <span id="6fd1">In some layout structs, I’ve used uintptr instead of
    actual pointers, again as a detterant. People who truly know how
    this works will be able to figure this out anyways.</span>
  - <span id="7b63">I’ve not written out all the layout structs.</span>
  - <span id="e438">Also memory alignment is tricky in the code segment,
    I haven’t gone into how to handle this. Someone who truly
    understands how this works should have no trouble doing it
    though.</span>
  - <span id="f97b">More cool things can be done using this technique,
    for instance, completely new go types can be created on the fly. But
    again, why would you do that? Use a dynamically typed language if
    you want that.</span>
  - <span id="f2e8">One possible excuse to go down this path is if
    you’re writing a dynamic debugger for golang.</span>

This guide is meant to be just a pointer to how some unnecessary and crazy 
things can be done with go.

