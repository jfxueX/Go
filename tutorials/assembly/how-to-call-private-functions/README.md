[How to call private functions (bind to hidden symbols) in GoLang](https://sitano.github.io/2016/04/28/golang-private/)
================================================================
28 Apr 2016

by [@JohnKoepi](https://twitter.com/JohnKoepi).


Names are as important in Go as in any other language. They even have semantic effect: the 
visibility of a name outside a package is determined by whether its first character is upper case.

Sometimes its necessary to overcome this limitation in order to organize your code better, or access 
some hidden functions in foreign packages.

These techniques are heavily used in golang source code, and this is the primary source of where it 
comes from. Its distinguishable lack of information there are on the internet on this topic.

From the good old days, there are 2 ways of achieving this bypassing compiler check: `cannot refer 
to unexported name pkg.symbol`:

-  the old one, currently not used - assembly level implicit linkage to needed symbols, referred as 
   `assembly stubs`, i.e. [go runtime, os/signal: use //go:linkname instead of assembly stubs to get 
   access to runtime functions][1];

-  the actual one - go compiler level support for link names redirection via `go:linkname`, since 
   11.11.14 [dev.cc code review 169360043: cmd/gc: changes for removing runtime C code (issue 169360043 
   by r…@golang.org)][2], mentioned on a github issue [cmd/compile: “missing function body” error when 
   using the //go:linkname compiler directive \#15006][3].

Using these techniques I have managed to bind to internal golang runtime schedule related functions 
to over use goroutines threads parking and internal locking mechanisms.

[1]: https://groups.google.com/forum/#!topic/%0Agolang-codereviews/J0HK9GLc76M
[2]: https://groups.google.com/forum/#!topic/%0Agolang-codereviews/5Ps_El_RpNE
[3]: https://github.com/golang/go/issues/15006

## Using `assembly stubs`

Idea is simple - *provide stubs in assembly with explicit JMPs to needed symbols.* Linker does not 
know anything about which symbols are exported and which not.

i.e. old version of `src/os/signal/sig.s`:

```asm
// Assembly to get into package runtime without using exported symbols.

// +build amd64 amd64p32 arm arm64 386 ppc64 ppc64le

#include "textflag.h"

#ifdef GOARCH_arm
#define JMP B
#endif
#ifdef GOARCH_ppc64
#define JMP BR
#endif
#ifdef GOARCH_ppc64le
#define JMP BR
#endif

TEXT ·signal_disable(SB),NOSPLIT,$0
    JMP runtime·signal_disable(SB)

TEXT ·signal_enable(SB),NOSPLIT,$0
    JMP runtime·signal_enable(SB)

TEXT ·signal_ignore(SB),NOSPLIT,$0
    JMP runtime·signal_ignore(SB)

TEXT ·signal_recv(SB),NOSPLIT,$0
    JMP runtime·signal_recv(SB)
```

and `signal_unix.go` binding:

```go
// +build darwin dragonfly freebsd linux nacl netbsd openbsd solaris windows

package signal

import (
    "os"
    "syscall"
)

// In assembly.
func signal_disable(uint32)
func signal_enable(uint32)
func signal_ignore(uint32)
func signal_recv() uint32
```

## Using `go:linkname`

In order to use this, the source file have to `import _ “unsafe”` package. To overcome `-complete` go 
compiler limitations one of possible solutions is to put empty assembly stub file near by the main 
source to disable this check.

i.e. `os/signal/sig.s`:

```asm
// The runtime package uses //go:linkname to push a few functions into this
// package but we still need a .s file so the Go tool does not pass -complete
// to the go tool compile so the latter does not complain about Go functions
// with no bodies.
```

The format of this instruction is `//go:linkname localname linkname`. Using this its possible to 
introduce new symbols for linkage (export), or bind to existing symbols (import).


### Export with `go:linkname`

A function implementation in `runtime/proc.go`

```go
...

//go:linkname sync_runtime_doSpin sync.runtime_doSpin
//go:nosplit
func sync_runtime_doSpin() {
    procyield(active_spin_cnt)
}
```

says explicitly to the compiler to add another name to the code which will be `runtime_doSpin` in 
`sync` package. And the `sync` reuses it in `sync/runtime.go` with simple:

```go
package sync

import "unsafe"

...

// runtime_doSpin does active spinning.
func runtime_doSpin()
```


### Import with `go:linkname`

A good example sits in `net/parse.go`:

```go
package net

import (
    ...
    _ "unsafe" // For go:linkname
)

...

// byteIndex is strings.IndexByte. It returns the index of the
// first instance of c in s, or -1 if c is not present in s.
// strings.IndexByte is implemented in  runtime/asm_$GOARCH.s
//go:linkname byteIndex strings.IndexByte
func byteIndex(s string, c byte) int
```

In order to use this technique:

1. Import `_ “unsafe”` package.
2. Give function definition without body, i.e. `func byteIndex(s string, c byte) int`
3. Put a `//go:linkname` instruction to the compiler right before the function definition, i.e. 
   `//go:linkname byteIndex strings.IndexByte`, where `byteIndex` is the local name, and 
   `strings.IndexByte` is remote name.
4. Provide `.s` file stub to allow compiler to bypass `-complete` check to allow partially defined functions.

### Example with `goparkunlock`

```go
package main

import (
    _ "unsafe"
    "fmt"
    "runtime/pprof"
    "os"
    "time"
)

// Event types in the trace, args are given in square brackets.
const (
    traceEvGoBlock        = 20 // goroutine blocks [timestamp, stack]
)

type mutex struct {
    // Futex-based impl treats it as uint32 key,
    // while sema-based impl as M* waitm.
    // Used to be a union, but unions break precise GC.
    key uintptr
}

//go:linkname lock runtime.lock
func lock(l *mutex)

//go:linkname unlock runtime.unlock
func unlock(l *mutex)

//go:linkname goparkunlock runtime.goparkunlock
func goparkunlock(lock *mutex, reason string, traceEv byte, traceskip int)

func main() {
    l := &mutex{}
    go func() {
        lock(l)
        goparkunlock(l, "xxx", traceEvGoBlock, 1)
    }()
    for {
        pprof.Lookup("goroutine").WriteTo(os.Stdout, 1)
        time.Sleep(time.Second * 1)
    }
}
```

### Sources

available at <https://github.com/sitano/gsysint>.


### Related Posts

-  ### [How to solve behemoth1 task or stack overflow showcase 28 Mar 2018](/2018/03/28/behemoth1/)

-  ### [How to solve behemoth0 task 25 Mar 2018](/2018/03/25/behemoth0/)

-  ### [How to read GoLang static single-assignment (SSA) form intermediate representation 18 Mar 2018](/2018/03/18/howto-read-gossa/)

