## Introduction
## 引言

> Go is a new language. Although it borrows ideas from existing languages, it has unusual properties 
> that make effective Go programs different in character from programs written in its relatives. A 
> straightforward translation of a C++ or Java program into Go is unlikely to produce a satisfactory 
> result—Java programs are written in Java, not Go. On the other hand, thinking about the problem from 
> a Go perspective could produce a successful but quite different program. In other words, to write Go 
> well, it's important to understand its properties and idioms. It's also important to know the 
> established conventions for programming in Go, such as naming, formatting, program construction, and 
> so on, so that programs you write will be easy for other Go programmers to understand.

Go 是一门全新的语言。尽管它从既有的语言中借鉴了许多理念，但其与众不同的特性， 使得使用 Go 编程在本质
上就不同于其它语言。将现有的 C++ 或 Java 程序直译为 Go 程序并不能令人满意——毕竟 Java 程序是用 Java 
编写的，而不是 Go。 另一方面，若从 Go 的角度去分析问题，你就能编写出同样可行但大不相同的程序。 换句
话说，要想将 Go 程序写得好，就必须理解其特性和风格。了解命名、格式化、 程序结构等既定规则也同样重
要，这样你编写的程序才能更容易被其他程序员所理解。

> This document gives tips for writing clear, idiomatic Go code. It augments the [language 
> specification](https://go-zh.org/ref/spec), [the Tour of Go](https://tour.golang.org/), and [How to 
> Write Go Code](https://go-zh.org/doc/code.html), all of which you should read first.

本文档就如何编写清晰、地道的 Go 代码提供了一些技巧。它是对 [语言规范](https://go-zh.org/ref/spec)、 
[Go 语言之旅](https://tour.golang.org/) 以及 [如何使用 Go 编程](https://go-zh.org/doc/code.html) 的
补充说明，因此我们建议您先阅读这些文档。

### Examples

### 示例

> The [Go package sources](https://golang.org/src/) are intended to serve not only as the core library 
> but also as examples of how to use the language. Moreover, many of the packages contain working, 
> self-contained executable examples you can run directly from the [golang.org](https://golang.org/) 
> web site, such as [this one](https://golang.org/pkg/strings/#example_Map) (if necessary, click on 
> the word"Example"to open it up). If you have a question about how to approach a problem or how 
> something might be implemented, the documentation, code and examples in the library can provide 
> answers, ideas and background.

[Go 包的源码](https://go-zh.org/src/pkg/) 不仅是核心库，同时也是学习如何使用 Go 语言的示例源码。 此
外，其中的一些包还包含了可工作的，独立的可执行示例，你可以直接在 [golang.org](https://golang.org/)  
网站上运行它们，比如 [这个例子](https://golang.org/pkg/strings/#example_Map) （单击文字 “示例” 来展
开它）。如果你有任何关于某些问题如何解决，或某些东西如何实现的疑问， 也可以从中获取相关的答案、思路
以及后台实现。
