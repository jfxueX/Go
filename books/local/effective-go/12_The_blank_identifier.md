## The blank identifier

## 空白标识符

We've mentioned the blank identifier a couple of times now, in the context of for [range loops]
(https://go-zh.org/doc/effective_go.html#for) and [maps](https://go-zh.org/doc/
effective_go.html#maps). The blank identifier can be assigned or declared with any value of any 
type, with the value discarded harmlessly. It's a bit like writing to the Unix /dev/null file: it 
represents a write-only value to be used as a place-holder where a variable is needed but the actual 
value is irrelevant. It has uses beyond those we've seen already.

我们在 [for-range](https://go-zh.org/doc/effective_go.html#for) 循环和 [映射](https://go-zh.org/doc/
effective_go.html#maps) 中提过几次空白标识符。 空白标识符可被赋予或声明为任何类型的任何值，而其值会
被无害地丢弃。它有点像 Unix 中的 /dev/null 文件：它表示只写的值，在需要变量但不需要实际值的地方用作
占位符。 我们在前面已经见过它的用法了。

### The blank identifier in multiple assignment

### 多重赋值中的空白标识符

The use of a blank identifier in a for range loop is a special case of a general situation: multiple 
assignment.

for range 循环中对空白标识符的用法是一种具体情况，更一般的情况即为多重赋值。

If an assignment requires multiple values on the left side, but one of the values will not be used 
by the program, a blank identifier on the left-hand-side of the assignment avoids the need to create 
a dummy variable and makes it clear that the value is to be discarded. For instance, when calling a 
function that returns a value and an error, but only the error is important, use the blank 
identifier to discard the irrelevant value.

若某次赋值需要匹配多个左值，但其中某个变量不会被程序使用， 那么用空白标识符来代替该变量可避免创建无
用的变量，并能清楚地表明该值将被丢弃。 例如，当调用某个函数时，它会返回一个值和一个错误，但只有错误
很重要， 那么可使用空白标识符来丢弃无关的值。

```go
if _, err := os.Stat(path); os.IsNotExist(err) {
	fmt.Printf("%s does not exist\n", path)
}
```
Occasionally you'll see code that discards the error value in order to ignore the error; this is 
terrible practice. Always check error returns; they're provided for a reason.

你偶尔会看见为忽略错误而丢弃错误值的代码，这是种糟糕的实践。请务必检查错误返回， 它们会提供错误的理
由。

```go
// Bad! This code will crash if path does not exist.
fi, _ := os.Stat(path)
if fi.IsDir() {
	fmt.Printf("%s is a directory\n", path)
}
```
```go
// 烂代码！若路径不存在，它就会崩溃。
fi, _ := os.Stat(path)
if fi.IsDir() {
	fmt.Printf("%s is a directory\n", path)
}
```
### Unused imports and variables

### 未使用的导入和变量

It is an error to import a package or to declare a variable without using it. Unused imports bloat 
the program and slow compilation, while a variable that is initialized but not used is at least a 
wasted computation and perhaps indicative of a larger bug. When a program is under active 
development, however, unused imports and variables often arise and it can be annoying to delete them 
just to have the compilation proceed, only to have them be needed again later. The blank identifier 
provides a workaround.

若导入某个包或声明某个变量而不使用它就会产生错误。未使用的包会让程序膨胀并拖慢编译速度， 而已初始化
但未使用的变量不仅会浪费计算能力，还有可能暗藏着更大的 Bug。 然而在程序开发过程中，经常会产生未使用
的导入和变量。虽然以后会用到它们， 但为了完成编译又不得不删除它们才行，这很让人烦恼。空白标识符就能
提供一个临时解决方案。

This half-written program has two unused imports (fmt and io) and an unused variable (fd), so it 
will not compile, but it would be nice to see if the code so far is correct.

这个写了一半的程序有两个未使用的导入（fmt 和 io）以及一个未使用的变量（fd），因此它不能编译， 但若到
目前为止代码还是正确的，我们还是很乐意看到它们的。

```go
package main

import (
    "fmt"
    "io"
    "log"
    "os"
)

func main() {
    fd, err := os.Open("test.go")
    if err != nil {
        log.Fatal(err)
    }
    // TODO: use fd.
}
```
To silence complaints about the unused imports, use a blank identifier to refer to a symbol from the 
imported package. Similarly, assigning the unused variable fd to the blank identifier will silence 
the unused variable error. This version of the program does compile.

要让编译器停止关于未使用导入的抱怨，需要空白标识符来引用已导入包中的符号。 同样，将未使用的变量 fd 
赋予空白标识符也能关闭未使用变量错误。 该程序的以下版本可以编译。

```go
package main

import (
    "fmt"
    "io"
    "log"
    "os"
)

var _ = fmt.Printf // For debugging; delete when done. // 用于调试，结束时删除。
var _ io.Reader    // For debugging; delete when done. // 用于调试，结束时删除。

func main() {
    fd, err := os.Open("test.go")
    if err != nil {
        log.Fatal(err)
    }
    // TODO: use fd.
    _ = fd
}
```
By convention, the global declarations to silence import errors should come right after the imports 
and be commented, both to make them easy to find and as a reminder to clean things up later.

按照惯例，我们应在导入并加以注释后，再使全局声明导入错误静默，这样可以让它们更易找到， 并作为以后清
理它的提醒。

### Import for side effect

### 为副作用而导入

An unused import like fmt or io in the previous example should eventually be used or removed: blank 
assignments identify code as a work in progress. But sometimes it is useful to import a package only 
for its side effects, without any explicit use. For example, during its init function, the net/http/
pprof package registers HTTP handlers that provide debugging information. It has an exported API, 
but most clients need only the handler registration and access the data through a web page. To 
import the package only for its side effects, rename the package to the blank identifier:

像前例中 fmt 或 io 这种未使用的导入总应在最后被使用或移除： 空白赋值会将代码标识为工作正在进行中。但
有时导入某个包只是为了其副作用， 而没有任何明确的使用。例如，在 net/http/pprof 包的 init 函数中记录
了 HTTP 处理程序的调试信息。它有个可导出的 API， 但大部分客户端只需要该处理程序的记录和通过 Web 页面
访问数据。欲导入一个只使用其副作用的包， 只需将该包重命名为空白标识符：

```go
import _ "net/http/pprof"
```
This form of import makes clear that the package is being imported for its side effects, because 
there is no other possible use of the package: in this file, it doesn't have a name. (If it did, and 
we didn't use that name, the compiler would reject the program.)

这种导入格式能明确表示该包是为其副作用而导入的，因为没有其它使用该包的可能： 在此文件中，它没有名
字。（若它有名字而我们没有使用，编译器就会拒绝该程序。）

### Interface checks

### 接口检查

As we saw in the discussion of [interfaces](https://go-zh.org/doc/
effective_go.html#interfaces_and_types) above, a type need not declare explicitly that it implements 
an interface. Instead, a type implements the interface just by implementing the interface's methods. 
In practice, most interface conversions are static and therefore checked at compile time. For 
example, passing an `*os.File` to a function expecting an io.Reader will not compile unless 
`*os.File` implements the io.Reader interface.

就像我们在前面 [接口](https://go-zh.org/doc/effective_go.html#interfaces_and_types) 中讨论的那样， 
一个类型无需显式地声明它实现了某个接口。取而代之，该类型只要实现了某个接口的方法， 其实就实现了该接
口。在实践中，大部分接口转换都是静态的，因此会在编译时检测。 例如，将一个 `*os.File` 传入一个接收 
io.Reader 的函数将不会被编译， 除非 `*os.File` 实现了 io.Reader 接口。

Some interface checks do happen at run-time, though. One instance is in the [encoding/json](https://
go-zh.org/pkg/encoding/json/) package, which defines a [Marshaler](Marshaler) interface. When the 
JSON encoder receives a value that implements that interface, the encoder invokes the value's 
marshaling method to convert it to JSON instead of doing the standard conversion. The encoder checks 
this property at run time with a [type assertion](https://go-zh.org/doc/
effective_go.html#interface_conversions) like:

尽管如此，有些接口检查会在运行时进行。例如，[encoding/json](https://go-zh.org/pkg/encoding/json/) 包
定义了一个 [Marshaler](Marshaler) 接口。当 JSON 编码器接收到一个实现了该接口的值，那么该编码器就会调
用该值的编组方法， 将其转换为 JSON，而非进行标准的类型转换。 编码器在运行时通过 [类型断言](https://
go-zh.org/doc/effective_go.html#interface_conversions) 检查其属性，就像这样：

```go
m, ok := val.(json.Marshaler)
```
If it's necessary only to ask whether a type implements an interface, without actually using the 
interface itself, perhaps as part of an error check, use the blank identifier to ignore the type-
asserted value:

若只需要判断某个类型是否是实现了某个接口，而不需要实际使用接口本身 （可能是错误检查部分），就使用空
白标识符来忽略类型断言的值：

```go
if _, ok := val.(json.Marshaler); ok {
	fmt.Printf("value %v of type %T implements json.Marshaler\n", val, val)
}
```
One place this situation arises is when it is necessary to guarantee within the package implementing 
the type that it actually satisfies the interface. If a type—for example, [json.RawMessage](https://
go-zh.org/pkg/encoding/json/#RawMessage)—needs a custom JSON representation, it should implement 
json.Marshaler, but there are no static conversions that would cause the compiler to verify this 
automatically. If the type inadvertently fails to satisfy the interface, the JSON encoder will still 
work, but will not use the custom implementation. To guarantee that the implementation is correct, a 
global declaration using the blank identifier can be used in the package:

当需要确保某个包中实现的类型一定满足该接口时，就会遇到这种情况。 若某个类型（例如 [json.RawMessage]
(https://go-zh.org/pkg/encoding/json/#RawMessage)） 需要一种定制的 JSON 表现时，它应当实现 
json.Marshaler， 不过现在没有静态转换可以让编译器去自动验证它。若该类型通过忽略转换失败来满足该接
口， 那么 JSON 编码器仍可工作，但它却不会使用定制的实现。为确保其实现正确， 可在该包中用空白标识符声
明一个全局变量：

```go
var _ json.Marshaler = (*RawMessage)(nil)
```
In this declaration, the assignment involving a conversion of a `*RawMessage` to a Marshaler 
requires that `*RawMessage` implements Marshaler, and that property will be checked at compile time. 
Should the json.Marshaler interface change, this package will no longer compile and we will be on 
notice that it needs to be updated.

在此声明中，我们调用了一个 `*RawMessage` 转换并将其赋予了 Marshaler，以此来要求 `*RawMessage` 实现 
Marshaler，这时其属性就会在编译时被检测。 若 json.Marshaler 接口被更改，此包将无法通过编译， 而我们
则会注意到它需要更新。

The appearance of the blank identifier in this construct indicates that the declaration exists only 
for the type checking, not to create a variable. Don't do this for every type that satisfies an 
interface, though. By convention, such declarations are only used when there are no static 
conversions already present in the code, which is a rare event.

在这种结构中出现空白标识符，即表示该声明的存在只是为了类型检查。 不过请不要为满足接口就将它用于任何
类型。作为约定， 仅当代码中不存在静态类型转换时才能这种声明，毕竟这是种罕见的情况。
