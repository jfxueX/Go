Writing a JIT compiler in Golang
================================

Sidhartha Mani

Dec 25, 2017


A JIT compiler (Just-in-Time) is any program that runs machine code generated during runtime. The 
difference between JIT code and other code (eg. fmt.Println) is that the JIT code is generated at 
runtime.

Programs written in Golang are statically typed, and compiled ahead of time. It might seem 
impossible to generate arbitrary code, let alone execute said code. However, it is possible to emit 
instructions into a running go process. This is done using Type Magic — the ability to convert any 
type to any other type.

As a side note, If you’re interested in hearing more about Type Magic, please leave a comment below 
and I’ll write about it next.


### Generating Code for x64

Machine code is a sequence of bytes that have a special meaning to the processor. The machine used 
to write this blog and test out the code uses a x64 processor, therefore I’ve used the x64 
instruction set.

This code will not run unless you’re running it on a x64 processor.


### Generating x64 code to print "Hello World!"

In order to print “Hello World” , a syscall should be made to instruct the processor to print data. 
The syscall to print data is [write(int fd, const void *buf, size_t count)](http://man7.org/linux/man-pages/man2/write.2.html).

The first parameter to this syscall is the location to write to, represented as a file descriptor. 
Printing output to the console is achieved by writing to the standard file descriptor called `stdout`. 
The file descriptor number for `stdout` is 1.

The second parameter is the location of the data that must be written. More information on this is 
provided in the next section.

The third operand is count — i.e. the number of bytes to write. In the case of “Hello World!”, the 
number of bytes to write is 12. In order to make the syscall, the three operands need to be saved in 
particular registers. Here’s a table showing the registers to save the operands in.

| Syscall #| Param 1| Param 2| Param 3| Param 4| Param 5| Param 6|
| -------- |------- | ------ | ------ | ------ | ------ | ------ |
| rax      |  rdi   |  rsi   |   rdx  |   r10  |   r8   |   r9   |

Putting all of this together, here’s a sequence of bytes that represent the instructions to 
initialize some of the registers.

```asm
0:  48 c7 c0 01 00 00 00    mov    $0x1, %rax 
7:  48 c7 c7 01 00 00 00    mov    $0x1, %rdi
e:  48 c7 c2 0c 00 00 00    mov    $0xc, %rdx
```

-  The first instruction sets `rax` to `1` — to denote the `write` syscall.
-  The second instruction sets `rdi` to `1` — to denote the file descriptor for `stdout`
-  The third instruction sets `rdx` to `12` to denote the number of bytes to print.
-  Location of the data is missing, and so is the actual call to `write`

In order to specify the location of data containing “Hello World!”, the data needs to have a 
location first — i.e. it needs to be stored somewhere in memory.

The byte sequence representing “Hello World!” is `48 65 6c 6c 6f 20 57 6f 72 6c 64 21`. This should be 
stored in a location where the processor will not try to execute it. Otherwise, the program will 
throw a segmentation fault error.

In this case, the data can be stored at the end of the executable instructions — i.e. after a return 
instruction. It is safe to store data after the return instruction because the processor "jumps" to 
a different address on encountering return and will not execute sequentially anymore.

Since the address past return is not known until the return instruction is laid out, a temporary 
place holder for it can be used and then replaced with the correct address once the address of the 
data is known. This is the exact procedure followed by linkers. The process of linking simply fills 
out these addresses to point to the correct data or function.

```asm
15: 48 8d 35 00 00 00 00    lea    0x0(%rip), %rsi      # 0x15
1c: 0f 05                   syscall
1e: c3                      ret
```

In the above code, the `lea` instruction to load the address of “Hello World!” is pointing to itself 
(to a location that is 0 bytes away from `rip`). This is because the data has not been stored yet and 
the address of the data is not known.

The syscall itself is represented by the byte sequence `0F 05`.

The data can now be stored, since the `return` instruction has been laid out.

```
1f: 48 65 6c 6c 6f 20 57 6f 72 6c 64 21   // Hello World!
```

With the whole program laid out, now we can update the `lea` instruction to point to the data. Here’s 
the updated code:

```asm
0:  48 c7 c0 01 00 00 00    mov    $0x1, %rax
7:  48 c7 c7 01 00 00 00    mov    $0x1, %rdi
e:  48 c7 c2 0c 00 00 00    mov    $0xc, %rdx
15: 48 8d 35 03 00 00 00    lea    0x3(%rip), %rsi        # 0x1f
1c: 0f 05                   syscall
1e: c3                      ret
1f: 48 65 6c 6c 6f 20 57 6f 72 6c 64 21   // Hello World! 
```

The above code can be represented as a slice of any primitive type in Golang.

A array/slice of `uint16` is a great choice because it can hold pairs of little-endian ordered words 
while still remaining readable. Here’s the `[]uint16` data structure holding the above program

```go
printFunction := []uint16{
    0x48c7, 0xc001, 0x0,                // mov $0x1, %rax
    0x48, 0xc7c7, 0x100, 0x0,           // mov $0x1, %rdi
    0x48c7, 0xc20c, 0x0,                // mov $0xc, %rdx
    0x48, 0x8d35, 0x400, 0x0,           // lea 0x4(%rip), %rsi
    0xf05,                              // syscall
    0xc3cc,                             // ret
    0x4865, 0x6c6c, 0x6f20,             // Hello_(whitespace)
    0x576f, 0x726c, 0x6421, 0xa,        // World!
} 
```

There is a slight deviation in the above bytes when compared to the bytes laid out above. This is 
because it is cleaner(easier to read and debug) to represent the data “Hello World!” when it is 
aligned to the start of a slice entry.

Therefore, I used the filler instruction `cc` instruction (no-op) to push the start of the data 
section to the next entry in the slice. I have also updated the `lea` to point to a location 4 bytes 
away to reflect this change.

*Note: You can find the syscall numbers for various syscalls at this [link][1].*

[1]: https://filippo.io/linux-syscall-table/


### Converting Slice to function

The instructions in the `[]uint16` data structure has to be converted into a function so that it can 
be called. The code below demonstrates this conversion.

```go
type printFunc func()

unsafePrintFunc := (uintptr)(unsafe.Pointer(&printFunction)) 
printer := *(*printFunc)(unsafe.Pointer(&unsafePrintFunc)) 
printer()
```

A Golang function value is just a pointer to a C function pointer ( *notice two levels of pointers* ). 
The conversion from slice to function begins by first extracting a pointer to the data structure 
which holds the executable code. This is stored in `unsafePrintFunc`. The pointer to `unsafePrintFunc` 
can be typecast into the desired function type.

This approach only works for functions without arguments or return values. A stack frame needs to be 
created for calling functions with arguments or return values. The function definition should always 
start with instructions to dynamically allocate the stack frame to support variadic functions. More 
information about different function types are available [here][2].

[2]: https://docs.google.com/document/d/1bMwCey-gmqZVTpRax-ESeVuZGmjwbocYs1iHplK-cjo/pub

*If you’d like me to write about generating more complex functions in Golang, please comment below.*


### Making the function executable

The above function will not actually run. This is because Golang stores all data structures in the 
data section of the binary. The data in this section has the [No-Execute][3] flag set on it, preventing 
it from being executed.

[3]: https://en.wikipedia.org/wiki/NX_bit

The data in the `printFunction` slice needs to be stored in a piece of memory that is executable. This 
can be achieved by either removing the No-Execute flag on the `printFunction` slice or by copying it 
to a location of memory that is executable.

In the code below, the data has been copied to a newly allocated piece of memory (using `mmap`) that 
is executable. This approach is preferable since *setting the no-execute flag is only possible on 
entire pages* — it is easily possible to unintentionally make other portions of the data section 
executable.

```go
  executablePrintFunc, err := syscall.Mmap(
     -1,
      0,
      128,  
      syscall.PROT_READ | syscall.PROT_WRITE | syscall.PROT_EXEC, 
      syscall.MAP_PRIVATE|syscall.MAP_ANONYMOUS
  )
  if err != nil {
    fmt.Printf("mmap err: %v", err)
  }
  j := 0
  for i := range printFunction {
    executablePrintFunc[j] = byte(printFunction[i] >> 8)
    executablePrintFunc[j+1] = byte(printFunction[i])
    j = j + 2
  }
```

The flag `syscall.PROT_EXEC` ensures that the newly allocated memory addresses are executable. 
Converting this data structure into a function will make it run smoothly. Here’s the complete code, 
which you can try out on your `x64` machine.

```go
package main

import (
	"fmt"
	"syscall"
	"unsafe"
)

type printFunc func()

func main() {
    printFunction := []uint16{
        0x48c7, 0xc001, 0x0,        // mov $0x1, %rax
        0x48, 0xc7c7, 0x100, 0x0,   // mov $0x1, %rdi
        0x48c7, 0xc20c, 0x0,        // mov $0xc, %rdx
        0x48, 0x8d35, 0x400, 0x0,   // lea 0x4(%rip), %rsi
        0xf05,                      // syscall
        0xc3cc,                     // ret
        0x4865, 0x6c6c, 0x6f20,     // Hello_(whitespace)
        0x576f, 0x726c, 0x6421, 0xa,// World!
    }
    
    executablePrintFunc, err := syscall.Mmap(
        -1,
        0,
        128,
        syscall.PROT_READ|syscall.PROT_WRITE|syscall.PROT_EXEC,
        syscall.MAP_PRIVATE|syscall.MAP_ANONYMOUS)
    if err != nil {
        fmt.Printf("mmap err: %v", err)
    }
    
    j := 0
    for i := range printFunction {
        executablePrintFunc[j] = byte(printFunction[i] >> 8)
        executablePrintFunc[j+1] = byte(printFunction[i])
        j = j + 2
    }
    
    type printFunc func()
    unsafePrintFunc := (uintptr)(unsafe.Pointer(&executablePrintFunc))
    printer := *(*printFunc)(unsafe.Pointer(&unsafePrintFunc))
    printer()
}
```

### Conclusion

Try out the source code above. Stay tuned for more deep dives into Golang!

Thanks to [Kynan Rilee](https://medium.com/@kynan.rilee).
