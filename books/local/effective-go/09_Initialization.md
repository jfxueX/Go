## Initialization
## 初始化

Although it doesn't look superficially very different from initialization in C or C++, 
initialization in Go is more powerful. Complex structures can be built during initialization and the 
ordering issues among initialized objects, even among different packages, are handled correctly.

尽管从表面上看，Go 的初始化过程与 C 或 C++ 相比并无太大差别，但它确实更为强大。 在初始化过程中，不仅
可以构建复杂的结构，还能正确处理不同包对象间的初始化顺序。

### Constants

### 常量

Constants in Go are just that—constant. They are created at compile time, even when defined as 
locals in functions, and can only be numbers, characters (runes), strings or booleans. Because of 
the compile-time restriction, the expressions that define them must be constant expressions, 
evaluatable by the compiler. For instance, 1<<3 is a constant expression, while math.Sin(math.Pi/4) 
is not because the function call to math.Sin needs to happen at run time.

Go 中的常量就是不变量。它们在编译时创建，即便它们可能是函数中定义的局部变量。 常量只能是数字、字符
（符文）、字符串或布尔值。由于编译时的限制， 定义它们的表达式必须也是可被编译器求值的常量表达式。例
如 1<<3 就是一个常量表达式，而 math.Sin(math.Pi/4) 则不是，因为对 math.Sin 的函数调用在运行时才会发
生。

In Go, enumerated constants are created using the iota enumerator. Since iota can be part of an 
expression and expressions can be implicitly repeated, it is easy to build intricate sets of values.

在 Go 中，枚举常量使用枚举器 iota 创建。由于 iota 可为表达式的一部分，而表达式可以被隐式地重复，这样
也就更容易构建复杂的值的集合了。

```go
type ByteSize float64

const (
    // 通过赋予空白标识符来忽略第一个值
    _           = iota // ignore first value by assigning to blank identifier
    KB ByteSize = 1 << (10 * iota)
    MB
    GB
    TB
    PB
    EB
    ZB
    YB
)
```
The ability to attach a method such as String to any user-defined type makes it possible for 
arbitrary values to format themselves automatically for printing. Although you'll see it most often 
applied to structs, this technique is also useful for scalar types such as floating-point types like 
ByteSize.

由于可将 String 之类的方法附加在用户定义的类型上， 因此它就为打印时自动格式化任意值提供了可能性，即
便是作为一个通用类型的一部分。 尽管你常常会看到这种技术应用于结构体，但它对于像 ByteSize 之类的浮点
数标量等类型也是有用的。

```go
func (b ByteSize) String() string {
    switch {
    case b >= YB:
        return fmt.Sprintf("%.2fYB", b/YB)
    case b >= ZB:
        return fmt.Sprintf("%.2fZB", b/ZB)
    case b >= EB:
        return fmt.Sprintf("%.2fEB", b/EB)
    case b >= PB:
        return fmt.Sprintf("%.2fPB", b/PB)
    case b >= TB:
        return fmt.Sprintf("%.2fTB", b/TB)
    case b >= GB:
        return fmt.Sprintf("%.2fGB", b/GB)
    case b >= MB:
        return fmt.Sprintf("%.2fMB", b/MB)
    case b >= KB:
        return fmt.Sprintf("%.2fKB", b/KB)
    }
    return fmt.Sprintf("%.2fB", b)
}
```
The expression YB prints as 1.00YB, while ByteSize(1e13) prints as 9.09TB.

表达式 YB 会打印出 1.00YB，而 ByteSize(1e13) 则会打印出 9.09TB。

The use here of Sprintf to implement ByteSize's String method is safe (avoids recurring 
indefinitely) not because of a conversion but because it calls Sprintf with %f, which is not a 
string format: Sprintf will only call the String method when it wants a string, and %f wants a 
floating-point value.

在这里用 Sprintf 实现 ByteSize 的 String 方法很安全（不会无限递归），这倒不是因为类型转换，而是它以 
%f 调用了 Sprintf，它并不是一种字符串格式：Sprintf 只会在它需要字符串时才调用 String 方法，而 %f 需
要一个浮点数值。

### Variables

### 变量

Variables can be initialized just like constants but the initializer can be a general expression 
computed at run time.

变量的初始化与常量类似，但其初始值也可以是在运行时才被计算的一般表达式。

```go
var (
	home   = os.Getenv("HOME")
	user   = os.Getenv("USER")
	gopath = os.Getenv("GOPATH")
)
```
### The init function

### init 函数

Finally, each source file can define its own niladic init function to set up whatever state is 
required. (Actually each file can have multiple init functions.) And finally means finally: init is 
called after all the variable declarations in the package have evaluated their initializers, and 
those are evaluated only after all the imported packages have been initialized.

最后，每个源文件都可以通过定义自己的无参数 init 函数来设置一些必要的状态。 （其实每个文件都可以拥有
多个 init 函数。）而它的结束就意味着初始化结束： 只有该包中的所有变量声明都通过它们的初始化器求值后 
init 才会被调用， 而那些 init 只有在所有已导入的包都被初始化后才会被求值。

Besides initializations that cannot be expressed as declarations, a common use of init functions is 
to verify or repair correctness of the program state before real execution begins.

除了那些不能被表示成声明的初始化外，init 函数还常被用在程序真正开始执行前，检验或校正程序的状态。

```go
func init() {
	if user == "" {
		log.Fatal("$USER not set")
	}
	if home == "" {
		home = "/home/" + user
	}
	if gopath == "" {
		gopath = home + "/go"
	}
	// gopath may be overridden by --gopath flag on command line.
	flag.StringVar(&gopath, "gopath", gopath, "override default GOPATH")
}
```
```go
func init() {
	if user == "" {
		log.Fatal("$USER not set")
	}
	if home == "" {
		home = "/home/" + user
	}
	if gopath == "" {
		gopath = home + "/go"
	}
	// gopath 可通过命令行中的 --gopath 标记覆盖掉。
	flag.StringVar(&gopath, "gopath", gopath, "override default GOPATH")
}
```
