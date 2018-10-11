## Errors

## 错误

Library routines must often return some sort of error indication to the caller. As mentioned 
earlier, Go's multivalue return makes it easy to return a detailed error description alongside the 
normal return value. It is good style to use this feature to provide detailed error information. For 
example, as we'll see, os.Open doesn't just return a nil pointer on failure, it also returns an 
error value that describes what went wrong.

库例程通常需要向调用者返回某种类型的错误提示。之前提到过，Go 语言的多值返回特性， 使得它在返回常规的
值时，还能轻松地返回详细的错误描述。使用这个特性来提供详细的错误信息是一种良好的风格。 例如，我们稍
后会看到， os.Open 在失败时不仅返回一个 nil 指针，还返回一个详细描述错误的 error 值。

By convention, errors have type error, a simple built-in interface.

按照约定，错误的类型通常为 error，这是一个内建的简单接口。

```go
type error interface {
	Error() string
}
```
A library writer is free to implement this interface with a richer model under the covers, making it 
possible not only to see the error but also to provide some context. As mentioned, alongside the 
usual `*os.File` return value, os.Open also returns an error value. If the file is opened 
successfully, the error will be nil, but when there is a problem, it will hold an os.PathError:

库的编写者通过更丰富的底层模型可以轻松实现这个接口，这样不仅能看见错误，还能提供一些上下文。前已述
及，除了通常的 `*os.File` 返回值， os.Open 还返回一个 error 值。若该文件被成功打开， error 值就是 
nil ，而如果出了问题，该值就是一个 os.PathError。

```go
// PathError records an error and the operation and
// file path that caused it.
type PathError struct {
	Op string    // "open", "unlink", etc.
	Path string  // The associated file.
	Err error    // Returned by the system call.
}

func (e *PathError) Error() string {
	return e.Op + " " + e.Path + ": " + e.Err.Error()
}
```
```go
// PathError 记录一个错误以及产生该错误的路径和操作。
type PathError struct {
	Op string    // "open"、"unlink" 等等。
	Path string  // 相关联的文件。
	Err error    // 由系统调用返回。
}

func (e *PathError) Error() string {
	return e.Op + " " + e.Path + ": " + e.Err.Error()
}
```
PathError's Error generates a string like this:

PathError 的 Error 会生成如下错误信息：

```go
open /etc/passwx: no such file or directory
```
Such an error, which includes the problematic file name, the operation, and the operating system 
error it triggered, is useful even if printed far from the call that caused it; it is much more 
informative than the plain "no such file or directory".

这种错误包含了出错的文件名、操作和触发的操作系统错误，即便在产生该错误的调用和输出的错误信息相距甚远
时，它也会非常有用，这比苍白的 “不存在该文件或目录” 更具说明性。

When feasible, error strings should identify their origin, such as by having a prefix naming the 
operation or package that generated the error. For example, in package image, the string 
representation for a decoding error due to an unknown format is "image: unknown format".

错误字符串应尽可能地指明它们的来源，例如产生该错误的包名前缀。例如在 image 包中，由于未知格式导致解
码错误的字符串为 “image: unknown format”。

Callers that care about the precise error details can use a type switch or a type assertion to look 
for specific errors and extract details. For PathErrors this might include examining the internal 
Err field for recoverable failures.

若调用者关心错误的完整细节，可使用类型选择或者类型断言来查看特定错误，并抽取其细节。 对于 
PathErrors，它应该还包含检查内部的 Err 字段以进行可能的错误恢复。

```go
for try := 0; try < 2; try++ {
	file, err = os.Create(filename)
	if err == nil {
		return
	}
	if e, ok := err.(*os.PathError); ok && e.Err == syscall.ENOSPC {
		deleteTempFiles()  // Recover some space.
		continue
	}
	return
}
```
```go
for try := 0; try < 2; try++ {
	file, err = os.Create(filename)
	if err == nil {
		return
	}
	if e, ok := err.(*os.PathError); ok && e.Err == syscall.ENOSPC {
		deleteTempFiles()  // 恢复一些空间。
		continue
	}
	return
}
```
The second if statement here is another [type assertion](https://go-zh.org/doc/
effective_go.html#interface_conversions). If it fails, ok will be false, and e will be nil. If it 
succeeds, ok will be true, which means the error was of type `*os.PathError`, and then so is e, 
which we can examine for more information about the error.

这里的第二条 if 是另一种 [类型断言](https://go-zh.org/doc/effective_go.html#interface_conversions)。
若它失败， ok 将为 false，而 e 则为 nil. 若它成功，ok 将为 true，这意味着该错误属于 `*os.PathError` 
类型，而 e 能够检测关于该错误的更多信息。

## Panic

## Panic

The usual way to report an error to a caller is to return an error as an extra return value. The 
canonical Read method is a well-known instance; it returns a byte count and an error. But what if 
the error is unrecoverable? Sometimes the program simply cannot continue.

向调用者报告错误的一般方式就是将 error 作为额外的值返回。 标准的 Read 方法就是个众所周知的实例，它返
回一个字节计数和一个 error。但如果错误时不可恢复的呢？有时程序就是不能继续运行。

For this purpose, there is a built-in function panic that in effect creates a run-time error that 
will stop the program (but see the next section). The function takes a single argument of arbitrary 
type—often a string—to be printed as the program dies. It's also a way to indicate that something 
impossible has happened, such as exiting an infinite loop.

为此，我们提供了内建的 panic 函数，它会产生一个运行时错误并终止程序 （但请继续看下一节）。该函数接受
一个任意类型的实参（一般为字符串），并在程序终止时打印。 它还能表明发生了意料之外的事情，比如从无限
循环中退出了。

```go
// A toy implementation of cube root using Newton's method.
func CubeRoot(x float64) float64 {
	z := x/3   // Arbitrary initial value
	for i := 0; i < 1e6; i++ {
		prevz := z
		z -= (z*z*z-x) / (3*z*z)
		if veryClose(z, prevz) {
			return z
		}
	}
	// A million iterations has not converged; something is wrong.
	panic(fmt.Sprintf("CubeRoot(%g) did not converge", x))
}
```
```go
// 用牛顿法计算立方根的一个玩具实现。
func CubeRoot(x float64) float64 {
	z := x/3   // 任意初始值
	for i := 0; i < 1e6; i++ {
		prevz := z
		z -= (z*z*z-x) / (3*z*z)
		if veryClose(z, prevz) {
			return z
		}
	}
	// 一百万次迭代并未收敛，事情出错了。
	panic(fmt.Sprintf("CubeRoot(%g) did not converge", x))
}
```
This is only an example but real library functions should avoid panic. If the problem can be masked 
or worked around, it's always better to let things continue to run rather than taking down the whole 
program. One possible counterexample is during initialization: if the library truly cannot set 
itself up, it might be reasonable to panic, so to speak.

这仅仅是个示例，实际的库函数应避免 panic。若问题可以被屏蔽或解决， 最好就是让程序继续运行而不是终止
整个程序。一个可能的反例就是初始化： 若某个库真的不能让自己工作，且有足够理由产生 Panic，那就由它去
吧。

```go
var user = os.Getenv("USER")

func init() {
	if user == "" {
		panic("no value for $USER")
	}
}
```
## Recover

## 恢复

When panic is called, including implicitly for run-time errors such as indexing a slice out of 
bounds or failing a type assertion, it immediately stops execution of the current function and 
begins unwinding the stack of the goroutine, running any deferred functions along the way. If that 
unwinding reaches the top of the goroutine's stack, the program dies. However, it is possible to use 
the built-in function recover to regain control of the goroutine and resume normal execution.

当 panic 被调用后（包括不明确的运行时错误，例如切片检索越界或类型断言失败）， 程序将立刻终止当前函数
的执行，并开始回溯 goroutine 的栈，运行任何被推迟的函数。 若回溯到达 goroutine 栈的顶端，程序就会终
止。不过我们可以用内建的 recover 函数来重新取回 goroutine 的控制权限并使其恢复正常执行。

A call to recover stops the unwinding and returns the argument passed to panic. Because the only 
code that runs while unwinding is inside deferred functions, recover is only useful inside deferred 
functions.

调用 recover 将停止回溯过程，并返回传入 panic 的实参。 由于在回溯时只有被推迟函数中的代码在运行，因
此 recover 只能在被推迟的函数中才有效。

One application of recover is to shut down a failing goroutine inside a server without killing the 
other executing goroutines.

recover 的一个应用就是在服务器中终止失败的 goroutine 而无需杀死其它正在执行的 goroutine。

```go
func server(workChan <-chan *Work) {
	for work := range workChan {
		go safelyDo(work)
	}
}

func safelyDo(work *Work) {
	defer func() {
		if err := recover(); err != nil {
			log.Println("work failed:", err)
		}
	}()
	do(work)
}
```
In this example, if do(work) panics, the result will be logged and the goroutine will exit cleanly 
without disturbing the others. There's no need to do anything else in the deferred closure; calling 
recover handles the condition completely.

在此例中，若 do(work) 触发了 Panic，其结果就会被记录， 而该 Go 程会被干净利落地结束，不会干扰到其它 
goroutine。我们无需在推迟的闭包中做任何事情， recover 会处理好这一切。

Because recover always returns nil unless called directly from a deferred function, deferred code 
can call library routines that themselves use panic and recover without failing. As an example, the 
deferred function in safelyDo might call a logging function before calling recover, and that logging 
code would run unaffected by the panicking state.

由于直接从被推迟函数中调用 recover 时不会返回 nil， 因此被推迟的代码能够调用本身使用了 panic 和 
recover 的库函数而不会失败。例如在 safelyDo 中，被推迟的函数可能在调用 recover 前先调用记录函数，而
该记录函数应当不受 Panic 状态的代码的影响。

With our recovery pattern in place, the do function (and anything it calls) can get out of any bad 
situation cleanly by calling panic. We can use that idea to simplify error handling in complex 
software. Let's look at an idealized version of a regexp package, which reports parsing errors by 
calling panic with a local error type. Here's the definition of Error, an error method, and the 
Compile function.

通过恰当地使用恢复模式，do 函数（及其调用的任何代码）可通过调用 panic 来避免更坏的结果。我们可以利用
这种思想来简化复杂软件中的错误处理。 让我们看看 regexp 包的理想化版本，它会以局部的错误类型调用 
panic 来报告解析错误。以下是一个 error 类型的 Error 方法和一个 Compile 函数的定义：

```go
// Error is the type of a parse error; it satisfies the error interface.
type Error string
func (e Error) Error() string {
	return string(e)
}

// error is a method of *Regexp that reports parsing errors by
// panicking with an Error.
func (regexp *Regexp) error(err string) {
	panic(Error(err))
}

// Compile returns a parsed representation of the regular expression.
func Compile(str string) (regexp *Regexp, err error) {
	regexp = new(Regexp)
	// doParse will panic if there is a parse error.
	defer func() {
		if e := recover(); e != nil {
			regexp = nil    // Clear return value.
			err = e.(Error) // Will re-panic if not a parse error.
		}
	}()
	return regexp.doParse(str), nil
}
```
```go
// Error 是解析错误的类型，它满足 error 接口。
type Error string
func (e Error) Error() string {
	return string(e)
}

// error 是 *Regexp 的方法，它通过用一个 Error 触发 Panic 来报告解析错误。
func (regexp *Regexp) error(err string) {
	panic(Error(err))
}

// Compile 返回该正则表达式解析后的表示。
func Compile(str string) (regexp *Regexp, err error) {
	regexp = new(Regexp)
	// doParse will panic if there is a parse error.
	defer func() {
		if e := recover(); e != nil {
			regexp = nil    // 清理返回值。
			err = e.(Error) // 若它不是解析错误，将重新触发 Panic。
		}
	}()
	return regexp.doParse(str), nil
}
```
If doParse panics, the recovery block will set the return value to nil—deferred functions can modify 
named return values. It will then check, in the assignment to err, that the problem was a parse 
error by asserting that it has the local type Error. If it does not, the type assertion will fail, 
causing a run-time error that continues the stack unwinding as though nothing had interrupted it. 
This check means that if something unexpected happens, such as an index out of bounds, the code will 
fail even though we are using panic and recover to handle parse errors.

若 doParse 触发了 Panic，恢复块会将返回值设为 nil —被推迟的函数能够修改已命名的返回值。在 err 的赋值
过程中， 我们将通过断言它是否拥有局部类型 Error 来检查它。若它没有， 类型断言将会失败，此时会产生运
行时错误，并继续栈的回溯，仿佛一切从未中断过一样。 该检查意味着若发生了一些像索引越界之类的意外，那
么即便我们使用了 panic 和 recover 来处理解析错误，代码仍然会失败。

With error handling in place, the error method (because it's a method bound to a type, it's fine, 
even natural, for it to have the same name as the builtin error type) makes it easy to report parse 
errors without worrying about unwinding the parse stack by hand:

通过适当的错误处理，error 方法（由于它是个绑定到具体类型的方法， 因此即便它与内建的 error 类型名字相
同也没有关系） 能让报告解析错误变得更容易，而无需手动处理回溯的解析栈：

```go
if pos == 0 {
	re.error("'*' illegal at start of expression")
}
```
Useful though this pattern is, it should be used only within a package. Parse turns its internal 
panic calls into error values; it does not expose panics to its client. That is a good rule to 
follow.

尽管这种模式很有用，但它应当仅在包内使用。Parse 会将其内部的 panic 调用转为 error 值，它并不会向调用
者暴露出 panic。这是个值得遵守的良好规则。

By the way, this re-panic idiom changes the panic value if an actual error occurs. However, both the 
original and new failures will be presented in the crash report, so the root cause of the problem 
will still be visible. Thus this simple re-panic approach is usually sufficient—it's a crash after 
all—but if you want to display only the original value, you can write a little more code to filter 
unexpected problems and re-panic with the original error. That's left as an exercise for the reader.

顺便一提，这种重新触发Panic的惯用法会在产生实际错误时改变Panic的值。 然而，不管是原始的还是新的错误
都会在崩溃报告中显示，因此问题的根源仍然是可见的。 这种简单的重新触发Panic的模型已经够用了，毕竟他只
是一次崩溃。 但若你只想显示原始的值，也可以多写一点代码来过滤掉不需要的问题，然后用原始值再次触发
Panic。 这里就将这个练习留给读者了。
