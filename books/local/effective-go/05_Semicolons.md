## Semicolons

## 分号

Like C, Go's formal grammar uses semicolons to terminate statements, but unlike in C, those 
semicolons do not appear in the source. Instead the lexer uses a simple rule to insert semicolons 
automatically as it scans, so the input text is mostly free of them.

和 C 一样，Go 的正式语法使用分号来结束语句；和 C 不同的是，这些分号并不在源码中出现。 取而代之，词法
分析器会使用一条简单的规则来自动插入分号，因此源码中基本就不用分号了。

The rule is this. If the last token before a newline is an identifier (which includes words like int 
and float64), a basic literal such as a number or string constant, or one of the tokens

规则是这样的：若在新行前的最后一个标记为标识符（包括 int 和 float64 这类的单词）、数值或字符串常量之
类的基本字面或以下标记之一

```go
break continue fallthrough return ++ -- ) }
```
the lexer always inserts a semicolon after the token. This could be summarized as, “if the newline 
comes after a token that could end a statement, insert a semicolon”.

则词法分析将始终在该标记后面插入分号。这点可以概括为： “如果新行前的标记为语句的末尾，则插入分号”。

A semicolon can also be omitted immediately before a closing brace, so a statement such as

分号也可在闭合的大括号之前直接省略，因此像

```go
	go func() { for { dst <- <-src } }()
```
needs no semicolons. Idiomatic Go programs have semicolons only in places such as for loop clauses, 
to separate the initializer, condition, and continuation elements. They are also necessary to 
separate multiple statements on a line, should you write code that way.

这样的语句无需分号。通常Go程序只在诸如 for 循环子句这样的地方使用分号， 以此来将初始化器、条件及增量
元素分开。如果你在一行中写多个语句，也需要用分号隔开。

One consequence of the semicolon insertion rules is that you cannot put the opening brace of a 
control structure (if, for, switch, or select) on the next line. If you do, a semicolon will be 
inserted before the brace, which could cause unwanted effects. Write them like this

警告：无论如何，你都不应将一个控制结构（if、for、switch 或 select）的左大括号放在下一行。如果这样
做，就会在大括号前面插入一个分号，这可能引起不需要的效果。 你应该这样写

```go
if i < f() {
	g()
}
```
not like this

而不是这样

```go
if i < f()  // wrong!
{           // wrong!
	g()
}
```

```go
if i < f()  // 错！
{           // 错！
	g()
}
```
