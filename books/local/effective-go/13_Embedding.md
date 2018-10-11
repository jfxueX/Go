## Embedding

## 内嵌

Go does not provide the typical, type-driven notion of subclassing, but it does have the ability to 
“borrow” pieces of an implementation by embedding types within a struct or interface.

Go 并不提供典型的，类型驱动的子类化概念，但通过将类型内嵌到结构体或接口中， 它就能 “借鉴” 部分实现。

Interface embedding is very simple. We've mentioned the io.Reader and io.Writer interfaces before; 
here are their definitions.

接口内嵌非常简单。我们之前提到过 io.Reader 和 io.Writer 接口，这里是它们的定义。

```go
type Reader interface {
	Read(p []byte) (n int, err error)
}

type Writer interface {
	Write(p []byte) (n int, err error)
}
```
The io package also exports several other interfaces that specify objects that can implement several 
such methods. For instance, there is io.ReadWriter, an interface containing both Read and Write. We 
could specify io.ReadWriter by listing the two methods explicitly, but it's easier and more 
evocative to embed the two interfaces to form the new one, like this:

io 包也导出了一些其它接口，以此来阐明对象所需实现的方法。 例如 io.ReadWriter 就是个包含 Read 和 
Write 的接口。我们可以通过显示地列出这两个方法来指明 io.ReadWriter， 但通过将这两个接口内嵌到新的接
口中显然更容易且更具启发性，就像这样：

```go
// ReadWriter is the interface that combines the Reader and Writer interfaces.
type ReadWriter interface {
	Reader
	Writer
}
```
```go
// ReadWriter 接口结合了 Reader 和 Writer 接口。
type ReadWriter interface {
	Reader
	Writer
}
```
This says just what it looks like: A ReadWriter can do what a Reader does and what a Writer does; it 
is a union of the embedded interfaces (which must be disjoint sets of methods). Only interfaces can 
be embedded within interfaces.

正如它看起来那样：ReadWriter 能够做任何 Reader 和 Writer 可以做到的事情，它是内嵌接口的联合体 （它们
必须是不相交的方法集）。只有接口能被嵌入到接口中。

The same basic idea applies to structs, but with more far-reaching implications. The bufio package 
has two struct types, bufio.Reader and bufio.Writer, each of which of course implements the 
analogous interfaces from package io. And bufio also implements a buffered reader/writer, which it 
does by combining a reader and a writer into one struct using embedding: it lists the types within 
the struct but does not give them field names.

同样的基本想法可以应用在结构体中，但其意义更加深远。bufio 包中有 bufio.Reader 和 bufio.Writer 这两个
结构体类型， 它们每一个都实现了与 io 包中相同意义的接口。此外，bufio 还通过结合 reader/writer 并将其
内嵌到结构体中，实现了带缓冲的 reader/writer：它在结构体中列出了这些类型，但并未给予它们字段名。

```go
// ReadWriter stores pointers to a Reader and a Writer.
// It implements io.ReadWriter.
type ReadWriter struct {
	*Reader  // *bufio.Reader
	*Writer  // *bufio.Writer
}
```
```go
// ReadWriter 存储了指向 Reader 和 Writer 的指针。
// 它实现了 io.ReadWriter。
type ReadWriter struct {
	*Reader  // *bufio.Reader
	*Writer  // *bufio.Writer
}
```
The embedded elements are pointers to structs and of course must be initialized to point to valid 
structs before they can be used. The ReadWriter struct could be written as

内嵌的元素为指向结构体的指针，当然它们在使用前必须被初始化为指向有效结构体的指针。 ReadWriter 结构体
可通过如下方式定义：

```go
type ReadWriter struct {
	reader *Reader
	writer *Writer
}
```
but then to promote the methods of the fields and to satisfy the io interfaces, we would also need 
to provide forwarding methods, like this:

但为了提升该字段的方法并满足 io 接口，我们同样需要提供转发的方法， 就像这样：

```go
func (rw *ReadWriter) Read(p []byte) (n int, err error) {
	return rw.reader.Read(p)
}
```
By embedding the structs directly, we avoid this bookkeeping. The methods of embedded types come 
along for free, which means that bufio.ReadWriter not only has the methods of bufio.Reader and 
bufio.Writer, it also satisfies all three interfaces: io.Reader, io.Writer, and io.ReadWriter.

而通过直接内嵌结构体，我们就能避免如此繁琐。 内嵌类型的方法可以直接引用，这意味着 bufio.ReadWriter 
不仅包括 bufio.Reader 和 bufio.Writer 的方法，它还同时满足下列三个接口： io.Reader、io.Writer 以及 
io.ReadWriter。

There's an important way in which embedding differs from subclassing. When we embed a type, the 
methods of that type become methods of the outer type, but when they are invoked the receiver of the 
method is the inner type, not the outer one. In our example, when the Read method of a 
bufio.ReadWriter is invoked, it has exactly the same effect as the forwarding method written out 
above; the receiver is the reader field of the ReadWriter, not the ReadWriter itself.

还有种区分内嵌与子类的重要手段。当内嵌一个类型时，该类型的方法会成为外部类型的方法， 但当它们被调用
时，该方法的接收者是内部类型，而非外部的。在我们的例子中，当 bufio.ReadWriter 的 Read 方法被调用时， 
它与之前写的转发方法具有同样的效果；接收者是 ReadWriter 的 reader 字段，而非 ReadWriter 本身。

Embedding can also be a simple convenience. This example shows an embedded field alongside a 
regular, named field.

内嵌同样可以提供便利。这个例子展示了一个内嵌字段和一个常规的命名字段。

```go
type Job struct {
	Command string
	*log.Logger
}
```
The Job type now has the Log, Logf and other methods of `*log.Logger`. We could have given the 
Logger a field name, of course, but it's not necessary to do so. And now, once initialized, we can 
log to the Job:

Job 类型现在有了 Log、Logf 和 `*log.Logger` 的其它方法。我们当然可以为 Logger 提供一个字段名，但完全
不必这么做。现在，一旦初始化后，我们就能记录 Job 了：

```go
job.Log("starting now...")
```
The Logger is a regular field of the Job struct, so we can initialize it in the usual way inside the 
constructor for Job, like this,

Logger 是 Job 结构体的常规字段， 因此我们可在 Job 的构造函数中，通过一般的方式来初始化它，就像这样：

```go
func NewJob(command string, logger *log.Logger) *Job {
	return &Job{command, logger}
}
```
or with a composite literal,

或通过复合字面：

```go
job := &Job{command, log.New(os.Stderr, "Job: ", log.Ldate)}
```
If we need to refer to an embedded field directly, the type name of the field, ignoring the package 
qualifier, serves as a field name, as it did in the Read method of our ReaderWriter struct. Here, if 
we needed to access the `*log.Logger` of a Job variable job, we would write job.Logger, which would 
be useful if we wanted to refine the methods of Logger.

若我们需要直接引用内嵌字段，可以忽略包限定名，直接将该字段的类型名作为字段名， 就像我们在 
ReaderWriter 结构体的 Read 方法中做的那样。 若我们需要访问 Job 类型的变量 job 的 `*log.Logger`， 可
以直接写作 job.Logger。若我们想精炼 Logger 的方法时， 这会非常有用。

```go
func (job *Job) Logf(format string, args ...interface{}) {
	job.Logger.Logf("%q: %s", job.Command, fmt.Sprintf(format, args...))
}

```
Embedding types introduces the problem of name conflicts but the rules to resolve them are simple. 
First, a field or method X hides any other item X in a more deeply nested part of the type. If 
log.Logger contained a field or method called Command, the Command field of Job would dominate it.

内嵌类型会引入命名冲突的问题，但解决规则却很简单。首先，字段或方法 X 会隐藏该类型中更深层嵌套的其它
项 X。若 log.Logger 包含一个名为 Command 的字段或方法，Job 的 Command 字段会覆盖它。

Second, if the same name appears at the same nesting level, it is usually an error; it would be 
erroneous to embed log.Logger if the Job struct contained another field or method called Logger. 
However, if the duplicate name is never mentioned in the program outside the type definition, it is 
OK. This qualification provides some protection against changes made to types embedded from outside; 
there is no problem if a field is added that conflicts with another field in another subtype if 
neither field is ever used.

其次，若相同的嵌套层级上出现同名冲突，通常会产生一个错误。若 Job 结构体中包含名为 Logger 的字段或方
法，再将 log.Logger 内嵌到其中的话就会产生错误。然而，若重名永远不会在该类型定义之外的程序中使用，那
就不会出错。 这种限定能够在外部嵌套类型发生修改时提供某种保护。 因此，就算添加的字段与另一个子类型中
的字段相冲突，只要这两个相同的字段永远不会被使用就没问题。
