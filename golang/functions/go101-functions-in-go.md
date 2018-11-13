# [Functions In Go][1]

[1]: https://go101.org/article/function.html
[2]: function-declarations-and-calls.html


[Function declarations and calls][2] have been explained before. The current 
article will touch more function related concpets and details in Go.

In fact, function is one kind of first-level citizen types in Go. In other 
words, we can use functions (but not including built-in functions) as values. 
Although Go is a static language, Go functions are very flexible. The feeling of 
using Go functions is much like using many dynamic languages.

There are some built-in functions in Go. These functions are declared in 
`builtin` and `unsafe` standard code packages. Built-in functions have some 
differences from custom declared functions. One difference is that built-in 
functions support generic parameters, but custom delcared ones don't (up to now, 
v1.11). More differences will be mentioned below.

### Function Signatures And Function Types

In Go, function is one kind of first-level citizen types. The literal of a 
function type is composed of the `func` keyword and a function signature 
literal. A function signature is composed of two type list, one is the input 
parameter type list, the other is the output result type lists. Parameter and 
result names can appear in function type and signature literals, but the names 
are not important.

We often use a function type literal as a function signature literal.

Here is a literal of a function type (and signature):

```go
func (a int, b string, c string) (x int, y int, z bool)
```

From the article [function declarations and calls][2], we have learned that 
consecutive variables with the same type can be declared together. So the above 
literal is equivalent to

```go
func (a int, b, c string) (x, y int, z bool)
```

As parameter names and result names are not important in the literals (as long 
as there are no duplicated non-blank names), the above ones are equivalent to 
the following one.

```go
func (x int, y, z string) (a, b int, c bool)
```

Variable names can be blank identifier `_`. The above ones are equivalent to the 
following one.

```go
func (_ int, _, _ string) (_, _ int, _ bool)
```

**The parameter names must be either all present or all absent**. The same rule is 
for result names. The above ones are equivalent to the following ones.

```go
func (int, string, string) (int, int, bool) // the standard form
func (a int, b string, c string) (int, int, bool)
func (x int, _ string, z string) (int, int, bool)
func (int, string, string) (x int, y int, z bool)
func (int, string, string) (a int, b int, _ bool)
```

All of the above function type literals denote the same (unnamed) function type.

Each parameter list must be enclosed in a `()` in a literal, even if the 
parameter list is blank. If a result list is blank, or has only one result and 
the name of the only result is absent, then the result list doesn't need to be 
enclosed in a `()`.

```go
// The following three function types are identical.
func () (x int)
func () (int)
func () int

// The following two function types are identical.
func (a int, b string) ()
func (a int, b string)
```

#### Variadic Parameters And Variadic Function Types

The last parameter of a function can be a variadic parameter. Each function can 
has at most one variadic parameter. To indicate the last parameter is variadic, 
just prefix three dots `...` to its type in its declaration. Example:

```go
func (values ...int64) (sum int64)
func (seperator string, tokens ...string) string
```

A function type with variadic parameter can be called a variadic function type. 
A variadic function type and a non-variadic function type are absolutely not 
identical.

#### Function Types Are Uncomprable Types

It has been [mentioned][3] several times in Go 101 that function types are 
uncomparable types.  Generally, function values can't be compared. But function 
values can be compared to the untyped bare `nil` identifier.

As function types are uncomprable types, they can't be used as the key types of 
map types.

[3]: type-system-overview.html#types-not-support-comparison

### Function Prototypes

A function prototype is composed of a function name and a function signature. 
Its literal is composed of the `func` keyword, a function name and a function 
signature.

A function prototype literal example:

```go
func Double(n int) (result int)
```

In other words, a function prototype is a function declaration without the body 
portion. A function declaration is composed of a function prototype and a 
function body.

### Variadic Function Declarations And Calls

General function declarations and calls have been explained in [function 
declarations and calls][2]. Here introduces how to declare and call variadic 
functions.

#### Variadic Function Declarations

Variadic function declarations are similar to general function declarations. 
Note, the variadic parameter of a variadic function will be treated as a slice 
within the body of the variadic function.

```go
// Sum and return the input numbers.
func Sum(values ...int64) (sum int64) {
    // The type of values is []int64.
    sum = 0
    for _, v := range values {
        sum += v
    }
    return
}

// An inefficient string concatenation function.
func Concat(seperator string, tokens ...string) string {
    // The type of tokens is []string.
    r := ""
    for i, t := range tokens {
        if i != 0 {
            r += seperator
        }
        r += t
    }
    return r
}
```

Below, for a variadic parameter declared with type as `...T`, we will say the 
type of the parameter is `[]T`.

In fact, the `Print`, `Println` and `Printf` functions in the `fmt` standard 
package are all variadic functions.

```go
func Print(a ...interface{}) (n int, err error)
func Printf(format string, a ...interface{}) (n int, err error)
func Println(a ...interface{}) (n int, err error)
```

The variadic parameter types are all `[]interface{}`. Interface types and values 
will be explained [interfaces in Go][4] later.

[4]: interface.html

#### Variadic Function Calls

There are two manners to pass arguments to a variadic parameter of type `[]T`:

1.  pass a slice value as the only argument. The slice must be assignable to 
    values of type `[]T`, and the slice must be followed by three dots `...`. 
    The passed slice is called as a variadic argument.

2.  pass zero or more arguments which are assignable to values of type `T`. 
    These arguments will be copied (or converted) as the elements of a new 
    allocated slice value of type `[]T`. The new allocated slice will be used as 
    the real variadic argument.

Note, the two manners can't be mixed in using.

An example program which uses some variadic function calls:

```go
package main

import "fmt"

func Sum(values ...int64) (sum int64) {
    sum = 0
    for _, v := range values {
        sum += v
    }
    return
}

func main() {
    a0 := Sum()
    a1 := Sum(2)
    a3 := Sum(2, 3, 5)
    // The above three lines are equivalent to
    // the following three lines.
    b0 := Sum([]int64{}...) // <=> Sum(nil...)
    b1 := Sum([]int64{2}...)
    b3 := Sum([]int64{2, 3, 5}...)
    fmt.Println(a0, a1, a3) // 0 2 10
    fmt.Println(b0, b1, b3) // 0 2 10
}
```

Another example:

```go
package main

import "fmt"

func Concat(seperator string, tokens ...string) (r string) {
    for i, t := range tokens {
        if i != 0 {
            r += seperator
        }
        r += t
    }
    return
}

func main() {
    tokens := []string{"Go", "C", "Rust"}
    langsA := Concat(",", tokens...)        // manner 1
    langsB := Concat(",", "Go", "C","Rust") // manner 2
    fmt.Println(langsA == langsB)           // true
}
```

The following example doesn't compile, for the two variadic function call 
manners are mixed.

```go
package main

func Sum(values ...int64) (sum int64) {
    // ...
    return
}

func Concat(seperator string, tokens ...string) (r string) {
    // ...
    return
}

func main() {
    // The following two lines both fail to compile,
    // for the same error: too many arguments in call.
    _ = Sum(2, []int64{3, 5}...)
    _ = Concat(",", "Go", []string{"C", "Rust"}...)
}
```

### More About Function Declarations And Calls

#### Functions Whose Names Can Be Duplicated

Generally, the names of the functions declared in the same code package can't be 
duplicated. But there are two exceptions.

1.  One exception is each code package can declare several functions with the 
    same name `init`. The prototypes of all the `init` functions must be `func 
    init()`. Each of these `init` functions will be called once and only once 
    when that code package is loaded at run time.

2.  The other exception is functions can be declared with names as the blank 
    identifier `_`, in which cases, the declared functions can never be called.

#### Some Function Calls Are Evaluated At Compile Time

Most function calls are evaluated at run time. But calls to the functions of the 
`unsafe` standard package are always evaluated at compile time. Calls to some 
other built-in functions, such as `len` and `cap`, [may be evaluated at either 
compile time or run time][4], depending on the passed arguments. The results of 
the function calls evaluated at compile time can be assigned to constants.

[4]: summaries.html#compile-time-evaluation
[5]: value-part.html#about-value-copy
[6]: https://golang.org/doc/asm

#### All Function Arguments Are Passed By Copy

Let's repeat it again, like all value assignments in Go, all function arguments 
are passed by copy in Go. When a value is copied, [only its direct part is 
copied][5].

#### Function Declarations Without Bodies

We can implement a function in [Go assembly][6]. Generally, Go assembly source 
files are stored in `*.a` files. A function implemented in Go assembly is still 
needed to be declared in a `*.go` file, but the only the prototype of the 
function is needed to be present. The body portion of the declaration of the 
function must be omitted in the `*.go` file.

#### Some Functions With Results Are Not Required To Return

If a function has return results, then the last statement in its declaration 
body must be a **[terminating statement][7]**. The function body is not required to 
contain a return statement. For example,

[7]: https://golang.org/ref/spec#Terminating_statements

```go
func fa() int {
    a:
    goto a
}

func fb() bool {
    for{}
}
```

#### The Results Of Most Function Calls Can Be Discarded

The return results of a custom function call can be all discarded together. The 
return results of calls to built-in functions, except `recover` and `copy`, 
can't be discarded, though they can be ignored by assigning them to some blank 
identifiers. Function calls whose results can't be discarded can't be used as 
deferred function calls or goroutine calls.

#### Use Function Calls As Expressions

A call to a function with single return result can always be used as a single 
value. For example, it can be nested in another function call as an argument, 
and can also be used as a single value to appear in any other expressions and 
statements.

If the return results of a call to a multi-result function are not discarded, 
then the call only can be used as a multi-valued expression in two scenarios.

1.  The call can be used in an assignment as source values. But the call can't 
    mix with other source values in the assignment.

2.  The call can be nested in another function call as arguments. But the call 
    can't mix with other arguments.

```go
package main

func HalfAndNegative(n int) (int, int) {
    return n/2, -n
}

func AddSub(a, b int) (int, int) {
    return a+b, a-b
}

func Dummy(values ...int) {}

func main() {
    // These lines compile okay.
    AddSub(HalfAndNegative(6))
    AddSub(AddSub(AddSub(7, 5)))
    AddSub(AddSub(HalfAndNegative(6)))
    Dummy(HalfAndNegative(6))
    _, _ = AddSub(7, 5)
    
    // These lines fail to compile.
    _, _, _ = 6, AddSub(7, 5)
    Dummy(AddSub(7, 5), 9)
    Dummy(AddSub(7, 5), HalfAndNegative(6))
}
```

Note, there are [exceptions][8] to the rules of using multi-valued expression.

[8]: exceptions.html#nest-function-calls

### Function Values

As above has mentioned, function types are one kind of types in Go. *A value of a 
function type is called a **function value** *. The zero values of function types are 
represented with the predeclared `nil`.

When we declare a custom function, we also declared an immutable function value 
in fact. The function value is identified by the function name. The type of the 
function value is represented as the literal by omitting the function name from 
the function prototype literal.

Note, built-in functions can't be used as values. `init` functions also can't be 
used as values.

Any function value can be invoked just like a declared function. It is fatal 
error to call a nil function to start a new goroutine. The fatal error is not 
recoverable and will make the whole program crash. For other situations, calls 
to nil function values will produce recoverable panics, including deferred 
function calls.

From the article [value parts][9], we know that non-nil function values are 
multi-part values. After one function value is assigned to another, the two 
functions share the same undrelying parts(s). In other words, the two functions 
represent the same internal function object. The effects of invoking two 
functions are the same.

[9]: value-part.html

An example:

```go
package main

import "fmt"

func Double(n int) int {
    return n + n
}

func Apply(n int, f func(int) int) int {
    return f(n) // the type of f is "func(int) int"
}

func main() {
    fmt.Printf("%T\n", Double) // func(int) int
    // Double = nil // error: Double is immutable.

    var f func(n int) int // default value is nil.
    f = Double
    g := Apply // let compile deduce the type of g
    fmt.Printf("%T\n", g) // func(int, func(int) int) int
    
    fmt.Println(f(9))         // 18
    fmt.Println(g(6, Double)) // 12
    fmt.Println(g(6, f))      // 12
}
```

In practice, we often assign anonymous functtions to function variables, so that 
we can call the anonymous functions multiple times.

```go
package main

import "fmt"

func main() {
    // This function returns a function (a closure).
    isMultipleOfX := func (x int) func(int) bool {
        return func(n int) bool {
            return n%x == 0
        }
    }
    
    var isMultipleOf3 = isMultipleOfX(3)
    var isMultipleOf5 = isMultipleOfX(5)
    fmt.Println(isMultipleOf3(6))  // true
    fmt.Println(isMultipleOf3(8))  // false
    fmt.Println(isMultipleOf5(10)) // true
    fmt.Println(isMultipleOf5(12)) // false
    
    isMultipleOf15 := func(n int) bool {
        return isMultipleOf3(n) && isMultipleOf5(n)
    }
    fmt.Println(isMultipleOf15(32)) // false
    fmt.Println(isMultipleOf15(60)) // true
}
```

All functions in Go can be viewed as closures. This is why user experiences of 
all kinds of Go functions are so uniform and why Go functions are as flexible as 
dynamic languages.


The ***Go 101*** project is hosted on both [Github][10] and [Gitlab][11]). 
Welcome to improve ***Go 101*** articles by submitting corrections for all kinds 
of mistakes, such as typos, grammar errors, wording inaccuracies, description 
flaws, code bugs and broken links.

Support Go 101 by playing [Tapir's games][12].

[10]: https://github.com/go101/go101
[11]: https://gitlab.com/Go101/go101
[12]: https://www.tapirgames.com
