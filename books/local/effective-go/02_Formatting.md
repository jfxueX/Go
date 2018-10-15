## Formatting

## 格式化

> Formatting issues are the most contentious but the least consequential. People can adapt to 
> different formatting styles but it's better if they don't have to, and less time is devoted to the 
> topic if everyone adheres to the same style. The problem is how to approach this Utopia without a 
> long prescriptive style guide.

格式化问题总是充满了争议，但却始终没有形成统一的定论。虽说人们可以适应不同的编码风格， 但抛弃这种适
应过程岂不更好？若所有人都遵循相同的编码风格，在这类问题上浪费的时间将会更少。 问题就在于如何实现这
种设想，而无需冗长的语言风格规范。

> With Go we take an unusual approach and let the machine take care of most formatting issues. The 
> gofmt program (also available as go fmt, which operates at the package level rather than source file 
> level) reads a Go program and emits the source in a standard style of indentation and vertical 
> alignment, retaining and if necessary reformatting comments. If you want to know how to handle some 
> new layout situation, run gofmt; if the answer doesn't seem right, rearrange your program (or file a 
> bug about gofmt), don't work around it.

在 Go 中我们另辟蹊径，让机器来处理大部分的格式化问题。gofmt 程序（也可用 go fmt，它以包为处理对象而
非源文件）将 Go 程序按照标准风格缩进、 对齐，保留注释并在需要时重新格式化。若你想知道如何处理一些新
的代码布局，请尝试运行 gofmt；若结果仍不尽人意，请重新组织你的程序（或提交有关 gofmt 的 Bug），而不
必为此纠结。

> As an example, there's no need to spend time lining up the comments on the fields of a structure. 
> Gofmt will do that for you. Given the declaration

举例来说，你无需花时间将结构体中的字段注释对齐，gofmt 将为你代劳。 假如有以下声明：

```go
type T struct {
	name string // name of the object
	value int // its value
}
```
```go
type T struct {
	name string // 对象名
	value int // 对象值
}
```

gofmt will line up the columns:

gofmt 会将它按列对齐为：

```go
type T struct {
	name    string // name of the object
	value   int    // its value
}
```
```go
type T struct {
	name    string // 对象名
	value   int    // 对象值
}
```

All Go code in the standard packages has been formatted with gofmt.

标准包中所有的 Go 代码都已经用 gofmt 格式化过了。

Some formatting details remain. Very briefly:

还有一些关于格式化的细节，它们非常简短：

```
Indentation
  We use tabs for indentation and gofmt emits them by default. Use spaces only if you must.
Line length
  Go has no line length limit. Don't worry about overflowing a punched card. If a line feels too 
long, wrap it and indent with an extra tab.
Parentheses
  Go needs fewer parentheses than C and Java: control structures (if, for, switch) do not have 
parentheses in their syntax. Also, the operator precedence hierarchy is shorter and clearer, so
    x<<8 + y<<16
  means what the spacing implies, unlike in the other languages.
```
```
缩进
  我们使用制表符（tab）缩进，gofmt 默认也使用它。在你认为确实有必要时再使用空格。
行的长度
  Go 对行的长度没有限制，别担心打孔纸不够长。如果一行实在太长，也可进行折行并插入适当的 tab 缩进。
括号
  比起 C 和 Java，Go 所需的括号更少：控制结构（if、for 和 switch）在语法上并不需要圆括号。此外，操作
符优先级处理变得更加简洁，因此
    x<<8 + y<<16
  正表述了空格符所传达的含义。
```
