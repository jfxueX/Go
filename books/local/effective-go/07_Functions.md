## Functions

## 函数

### Multiple return values

### 多值返回

> One of Go's unusual features is that functions and methods can return multiple values. This form can 
> be used to improve on a couple of clumsy idioms in C programs: in-band error returns such as -1 for 
> EOF and modifying an argument passed by address.

Go 与众不同的特性之一就是函数和方法可返回多个值。这种形式可以改善 C 中一些笨拙的习惯：将错误值返回
（例如用 -1 表示 EOF）并修改通过地址传入的实参。

> In C, a write error is signaled by a negative count with the error code secreted away in a volatile 
> location. In Go, Write can return a count and an error: “Yes, you wrote some bytes but not all of 
> them because you filled the device”. The signature of the Write method on files from package os is:

在 C 中，写入操作发生的错误会用一个负的写入字节数标记，而错误码会隐藏在某个不确定的位置。而在 Go 
中，Write 会返回写入的字节数以及一个错误：“是的，您写入了一些字节，但并未全部写入，因为设备已满”。在 
os 包中，File.Write 的签名为：

```go
func (file *File) Write(b []byte) (n int, err error)
```

> and as the documentation says, it returns the number of bytes written and a non-nil error when n != 
> len(b). This is a common style; see the section on error handling for more examples.

正如文档所述，它返回写入的字节数，并在 <code>n != len(b)</code> 时返回一个非 nil 的 error 错误值。这是一种常见的
编码风格，更多示例见错误处理一节。

> A similar approach obviates the need to pass a pointer to a return value to simulate a reference 
> parameter. Here's a simple-minded function to grab a number from a position in a byte slice, 
> returning the number and the next position.

我们可以采用一种简单的方法。来避免为模拟引用参数而传入指针。以下简单的函数可从字节数组中的特定位置
获取一个数值，并返回该数值和下一个位置。

```go
func nextInt(b []byte, i int) (int, int) {
	for ; i < len(b) && !isDigit(b[i]); i++ {
	}
	x := 0
	for ; i < len(b) && isDigit(b[i]); i++ {
		x = x*10 + int(b[i]) - '0'
	}
	return x, i
}
```
> You could use it to scan the numbers in an input slice b like this:

你可以像下面这样，通过它扫描输入的切片 b 来获取数字。

```go
for i := 0; i < len(b); {
	x, i = nextInt(b, i)
	fmt.Println(x)
}
```
### Named result parameters

### 可命名结果形参

> The return or result "parameters" of a Go function can be given names and used as regular variables, 
> just like the incoming parameters. When named, they are initialized to the zero values for their 
> types when the function begins; if the function executes a return statement with no arguments, the 
> current values of the result parameters are used as the returned values.

Go 函数的返回值或结果“形参”可被命名，并作为常规变量使用，就像传入的形参一样。命名后，一旦该函数开
始执行，它们就会被初始化为与其类型相应的零值；若该函数执行一条 return 语句未带实参（语句为 `return`），
则结果形参的当前值被用作返回值。

> The names are not mandatory but they can make code shorter and clearer: they're documentation. If we 
> name the results of nextInt it becomes obvious which returned int is which.

此名称不是强制性的，但它们能使代码更加简短清晰：它们就是文档。若我们命名了 nextInt 的返回值，那么它返
回的哪一个 int 是什么就很清楚了。

```go
func nextInt(b []byte, pos int) (value, nextPos int) {
```

> Because named results are initialized and tied to an unadorned return, they can simplify as well as 
> clarify. Here's a version of io.ReadFull that uses them well:

由于被命名的结果已经初始化，且已绑定到不带参数的 `return`，它们就能让代码简单而清晰。下面的 io.ReadFull 
就是个很好的例子：

```go
func ReadFull(r Reader, buf []byte) (n int, err error) {
	for len(buf) > 0 && err == nil {
		var nr int
		nr, err = r.Read(buf)
		n += nr
		buf = buf[nr:]
	}
	return
}
```
### Defer

> Go's defer statement schedules a function call (the deferred function) to be run immediately before 
> the function executing the defer returns. It's an unusual but effective way to deal with situations 
> such as resources that must be released regardless of which path a function takes to return. The 
> canonical examples are unlocking a mutex or closing a file.

Go 的 defer 语句用于预设一个函数调用（即推迟执行函数），正在执行 defer 的函数在返回之前会立即执行该
延迟函数。它不同寻常，但却是处理一些事情的有效方式，例如无论以何种路径返回，都必须释放资源的函数。典
型的例子就是解锁互斥和关闭文件。

> ```go
> // Contents returns the file's contents as a string.
> func Contents(filename string) (string, error) {
> 	f, err := os.Open(filename)
> 	if err != nil {
> 		return "", err
> 	}
> 	defer f.Close()  // f.Close will run when we're finished.
> 
> 	var result []byte
> 	buf := make([]byte, 100)
> 	for {
> 		n, err := f.Read(buf[0:])
> 		result = append(result, buf[0:n]...) // append is discussed later.
> 		if err != nil {
> 			if err == io.EOF {
> 				break
> 			}
> 			return "", err  // f will be closed if we return here.
> 		}
> 	}
> 	return string(result), nil // f will be closed if we return here.
> }
> ```

```go
// Contents 将文件的内容作为字符串返回。
func Contents(filename string) (string, error) {
	f, err := os.Open(filename)
	if err != nil {
		return "", err
	}
	defer f.Close()  // f.Close 会在我们结束后运行。

	var result []byte
	buf := make([]byte, 100)
	for {
		n, err := f.Read(buf[0:])
		result = append(result, buf[0:n]...) // append 将在后面讨论。
		if err != nil {
			if err == io.EOF {
				break
			}
			return "", err  // 我们在这里返回后，f 就会被关闭。
		}
	}
	return string(result), nil // 我们在这里返回后，f 就会被关闭。
}
```
> Deferring a call to a function such as Close has two advantages. First, it guarantees that you will 
> never forget to close the file, a mistake that's easy to make if you later edit the function to add 
> a new return path. Second, it means that the close sits near the open, which is much clearer than 
> placing it at the end of the function.

推迟（诸如 Close 之类的）函数调用有两点好处：首先，它能确保你不会忘记关闭文件，这是以后再为该函数添
加新的返回路径时很容易犯的错误。其次，它意味着 “关闭”离 “打开”很近，这总比将它放在函数结尾处要清晰明
了。

> The arguments to the deferred function (which include the receiver if the function is a method) are 
> evaluated when the defer executes, not when the call executes. Besides avoiding worries about 
> variables changing values as the function executes, this means that a single deferred call site can 
> defer multiple function executions. Here's a silly example.

传给被推迟函数的实参（如果该函数为方法则还包括接收者）在 `defer` 执行时就会被求值，而不是在调用执行
时才求值。这样不仅无需担心变量值在函数执行时被改变，同时还意味着单个被推迟的调用可推迟多个函数的执
行。下面是个简单的例子。

```go
for i := 0; i < 5; i++ {
	defer fmt.Printf("%d ", i)
}
```

> Deferred functions are executed in LIFO order, so this code will cause 4 3 2 1 0 to be printed when 
> the function returns. A more plausible example is a simple way to trace function execution through 
> the program. We could write a couple of simple tracing routines like this:

被推迟的函数按照后进先出（LIFO）的顺序执行，因此以上代码在函数返回时会打印 4 3 2 1 0。一个更具实际意
义的例子是通过一种简单的方法，用程序来跟踪函数的执行。我们可以编写一对简单的跟踪例程：

> ```go
> func trace(s string)   { fmt.Println("entering:", s) }
> func untrace(s string) { fmt.Println("leaving:", s) }
> 
> // Use them like this:
> func a() {
> 	trace("a")
> 	defer untrace("a")
> 	// do something....
> }
> ```

```go
func trace(s string)   { fmt.Println("entering:", s) }
func untrace(s string) { fmt.Println("leaving:", s) }

// 像这样使用它们：
func a() {
	trace("a")
	defer untrace("a")
	// 做一些事情....
}
```

> We can do better by exploiting the fact that arguments to deferred functions are evaluated when the 
> defer executes. The tracing routine can set up the argument to the untracing routine. This example:

我们可以充分利用这个特点，即被推迟函数的实参在 defer 执行时就会被求值。跟踪例程可针对反跟踪例程设置
实参。以下例子：

```go
func trace(s string) string {
	fmt.Println("entering:", s)
	return s
}

func un(s string) {
	fmt.Println("leaving:", s)
}

func a() {
	defer un(trace("a"))
	fmt.Println("in a")
}

func b() {
	defer un(trace("b"))
	fmt.Println("in b")
	a()
}

func main() {
	b()
}
```
> prints

会打印

```go
entering: b
in b
entering: a
in a
leaving: a
leaving: b
```

> For programmers accustomed to block-level resource management from other languages, defer may seem 
> peculiar, but its most interesting and powerful applications come precisely from the fact that it's 
> not block-based but function-based. In the section on panic and recover we'll see another example of 
> its possibilities.

对于习惯其它语言中块级资源管理的程序员，defer 似乎有点怪异，但它最有趣而强大的应用恰恰来自于其基于
函数而非块的特点。在 panic 和 recover 这两节中，我们将看到关于它可能性的其它例子。
