## Control structures

## 控制结构

> The control structures of Go are related to those of C but differ in important ways. There is no do 
> or while loop, only a slightly generalized for; switch is more flexible; if and switch accept an 
> optional initialization statement like that of for; break and continue statements take an optional 
> label to identify what to break or continue; and there are new control structures including a type 
> switch and a multiway communications multiplexer, select. The syntax is also slightly different: 
> there are no parentheses and the bodies must always be brace-delimited.

Go 中的结构控制与 C 有许多相似之处，但其不同之处才是独到之处。Go 不再使用 do 或 while 循环，只有一
个更通用的 for；switch 要更灵活一点；if 和 switch 像 for 一样可接受可选的初始化语句；break 和 
continue 语句带一个可选的标签来标识 break 或 continue 什么；此外，还有包含一个类型选择和一个多路通信
复用器的新控制结构：select。其语法也有些许不同：没有圆括号，而其主体必须始终使用大括号括住。

### If

> In Go a simple if looks like this:

在 Go 中，一个简单的 if 语句看起来像这样：

```go
if x > 0 {
	return y
}
```
> Mandatory braces encourage writing simple if statements on multiple lines. It's good style to do so 
> anyway, especially when the body contains a control statement such as a return or break.

强制的大括号促使你将简单的 if 语句分成多行。特别是在主体中包含 return 或 break 等控制语句时，这种编
码风格的好处一比便知。

> Since if and switch accept an initialization statement, it's common to see one used to set up a 
> local variable.

由于 if 和 switch 可接受初始化语句，因此用它们来设置局部变量十分常见。

```go
if err := file.Chmod(0664); err != nil {
	log.Print(err)
	return err
}
```
> In the Go libraries, you'll find that when an if statement doesn't flow into the next statement—that 
> is, the body ends in break, continue, goto, or return—the unnecessary else is omitted.

在 Go 的库中，你会发现若 if 语句不会执行到下一条语句时，亦即其执行体以 break、continue、goto 或 
return 结束时，不必要的 else 会被省略。

```go
f, err := os.Open(name)
if err != nil {
	return err
}
codeUsing(f)
```

> This is an example of a common situation where code must guard against a sequence of error 
> conditions. The code reads well if the successful flow of control runs down the page, eliminating 
> error cases as they arise. Since error cases tend to end in return statements, the resulting code 
> needs no else statements.

下例是一种常见的情况，代码必须防范一系列的错误条件。若控制流成功继续，则说明程序已排除错误。由于出
错时将以 return 结束，之后的代码也就无需 else 了。

```go
f, err := os.Open(name)
if err != nil {
	return err
}
d, err := f.Stat()
if err != nil {
	f.Close()
	return err
}
codeUsing(f, d)
```
### Redeclaration and reassignment

### 重新声明与再次赋值

An aside: The last example in the previous section demonstrates a detail of how the := short 
declaration form works. The declaration that calls os.Open reads,

题外话：上一节中最后一个示例展示了短声明 `:=` 如何使用。调用了 os.Open 的声明为

```go
f, err := os.Open(name)
```
> This statement declares two variables, f and err. A few lines later, the call to f.Stat reads,

该语句声明了两个变量 f 和 err。在几行之后，又通过

```go
d, err := f.Stat()
```
> which looks as if it declares d and err. Notice, though, that err appears in both statements. This 
> duplication is legal: err is declared by the first statement, but only re-assigned in the second. 
> This means that the call to f.Stat uses the existing err variable declared above, and just gives it 
> a new value.

调用了 f.Stat。它看起来似乎是声明了 d 和 err。注意，尽管两个语句中都出现了 err，但这种重复仍然是合
法的：err 在第一条语句中被声明，但在第二条语句中只是被再次赋值罢了。也就是说，调用 f.Stat 使用的是前
面已经声明的 err，它只是被重新赋值了而已。

> In a := declaration a variable v may appear even if it has already been declared, provided:

在满足下列条件时，已被声明的变量 v 可出现在 := 声明中：

> + this declaration is in the same scope as the existing declaration of v (if v is already declared 
> in an outer scope, the declaration will create a new variable §),
> + the corresponding value in the initialization is assignable to v, and
> + there is at least one other variable in the declaration that is being declared anew.


+ 本次声明与已声明的 v 处于同一作用域中（若 v 已在外层作用域中声明过，则此次声明会创建一个新的变量 
§），
+ 在初始化中与其类型相应的值才能赋予 v，且
+ 在此次声明中至少另有一个变量是新声明的（如上面的例子）。

> This unusual property is pure pragmatism, making it easy to use a single err value, for example, in 
> a long if-else chain. You'll see it used often.

这个特性简直就是纯粹的实用主义体现，它使得我们可以很方便地只使用一个 err 值，例如，在一个相当长的 
if-else 语句链中，你会发现它用得很频繁。

> § It's worth noting here that in Go the scope of function parameters and return values is the same 
> as the function body, even though they appear lexically outside the braces that enclose the body.

§ 值得一提的是，即便 Go 中的函数形参和返回值在词法上处于大括号之外，但它们的作用域和该函数体仍然相
同。

### For

> The Go for loop is similar to—but not the same as—C's. It unifies for and while and there is no do-
> while. There are three forms, only one of which has semicolons.

Go 的 for 循环类似于 C，但却不尽相同。它统一了 for 和 while，不再有 do-while 了。它有三种形式，但只
有一种需要分号。

> ```go
> // Like a C for
> for init; condition; post { }
> 
> // Like a C while
> for condition { }
> 
> // Like a C for(;;)
> for { }
> ```

```go
// 如同 C 的 for 循环
for init; condition; post { }

// 如同 C 的 while 循环
for condition { }

// 如同 C 的 for(;;) 循环
for { }
```
> Short declarations make it easy to declare the index variable right in the loop.

简短声明（`i := 0`）能让我们更容易在循环中声明下标变量：

```go
sum := 0
for i := 0; i < 10; i++ {
	sum += i
}
```
> If you're looping over an array, slice, string, or map, or reading from a channel, a range clause 
> can manage the loop.

若你想遍历数组、切片、字符串或者映射，或从信道中读取消息，range 子句能够帮你轻松实现循环。

```go
for key, value := range oldMap {
	newMap[key] = value
}
```

> If you only need the first item in the range (the key or index), drop the second:

若你只需要该遍历中的第一个项（键值或下标），去掉第二个就行了：

```go
for key := range m {
	if key.expired() {
		delete(m, key)
	}
}
```

> If you only need the second item in the range (the value), use the blank identifier, an underscore, 
> to discard the first:

若你只需要该遍历中的第二个项（值），请使用空白标识符，即下划线来丢弃第一个值：

```go
sum := 0
for _, value := range array {
	sum += value
}
```

> The blank identifier has many uses, as described in a later section.

空白标识符还有多种用法，它会在后面的小节中描述。

> For strings, the range does more work for you, breaking out individual Unicode code points by 
> parsing the UTF-8. Erroneous encodings consume one byte and produce the replacement rune U+FFFD. 
> (The name (with associated builtin type) rune is Go terminology for a single Unicode code point. See 
> the language specification for details.) The loop

对于字符串，range 能够提供更多便利。它能通过解析 UTF-8，将每个独立的 Unicode 码点分离出来。错误的编
码将占用一个字节，并以符文 U+FFFD 来代替。（名称 “符文”和内建类型 rune 是 Go 对单个 Unicode 码点的
称谓。详情见语言规范）。循环

> ```go
> for pos, char := range "日本\x80語" { // \x80 is an illegal UTF-8 encoding
> 	fmt.Printf("character %#U starts at byte position %d\n", char, pos)
> }
> ```
> prints
> 
> ```go
> character U+65E5 '日' starts at byte position 0
> character U+672C '本' starts at byte position 3
> character U+FFFD '�' starts at byte position 6
> character U+8A9E '語' starts at byte position 7
> ```

```go
for pos, char := range "日本\x80語" { // \x80 是个非法的UTF-8编码
	fmt.Printf("字符 %#U 始于字节位置 %d\n", char, pos)
}
```
将打印

```go
字符 U+65E5 '日' 始于字节位置 0
字符 U+672C '本' 始于字节位置 3
字符 U+FFFD '�' 始于字节位置 6
字符 U+8A9E '語' 始于字节位置 7
```

> Finally, Go has no comma operator and ++ and -- are statements not expressions. Thus if you want to 
> run multiple variables in a for you should use <b>parallel assignment</b> (although that precludes ++ and 
> --).

最后，Go 没有逗号操作符，而 ++ 和 -- 为语句而非表达式。因此，若你想要在 for 中使用多个变量，应采用
平行赋值的方式 （尽管这排斥了 ++ 和 --）。

> ```go
> // Reverse a
> for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 {
> 	a[i], a[j] = a[j], a[i]
> }
> ```

```go
// 反转 a
for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 {
	a[i], a[j] = a[j], a[i]
}
```

### Switch

> Go's switch is more general than C's. The expressions need not be constants or even integers, the 
> cases are evaluated top to bottom until a match is found, and if the switch has no expression it 
> switches on true. It's therefore possible—and idiomatic—to write an if-else-if-else chain as a 
> switch.

Go 的 switch 比 C 的更通用。其表达式无需为常量或整数，case 语句会自上而下逐一进行求值直到匹配为止。
若 switch 子句后面不带表达式，它将匹配 true，因此，我们可以将 if-else-if-else 链写成一个 switch，这
也更符合 Go 的风格。

```go
func unhex(c byte) byte {
	switch {
	case '0' <= c && c <= '9':
		return c - '0'
	case 'a' <= c && c <= 'f':
		return c - 'a' + 10
	case 'A' <= c && c <= 'F':
		return c - 'A' + 10
	}
	return 0
}
```
> There is no automatic fall through, but cases can be presented in comma-separated lists.

switch 并不会自动下溯，但 case 可通过逗号分隔来列举相同的处理条件。

```go
func shouldEscape(c byte) bool {
	switch c {
	case ' ', '?', '&', '=', '#', '+', '%':
		return true
	}
	return false
}
```

> Although they are not nearly as common in Go as some other C-like languages, break statements can be 
> used to terminate a switch early. Sometimes, though, it's necessary to break out of a surrounding 
> loop, not the switch, and in Go that can be accomplished by putting a label on the loop 
> and "breaking" to that label. This example shows both uses.

尽管它们在 Go 中不像其他类 C 语言中那样常见，但 break 语句可以用来提前终止 switch。有时候有必要中断
周围的循环，而不是 switch，在 Go 中，可以通过在循环上放置一个标签，并 break 到该标签来实现。下面的例
子展示了二者的用法。

```go
Loop:
	for n := 0; n < len(src); n += size {
		switch {
		case src[n] < sizeOne:
			if validateOnly {
				break
			}
			size = 1
			update(src[n])

		case src[n] < sizeTwo:
			if n+1 >= len(src) {
				err = errShortInput
				break Loop
			}
			if validateOnly {
				break
			}
			size = 2
			update(src[n] + src[n+1]<<shift)
		}
	}
```

> Of course, the continue statement also accepts an optional label but it applies only to loops.

当然，continue 语句也能接受一个可选的标签，不过它只能在循环中使用。

> To close this section, here's a comparison routine for byte slices that uses two switch statements:
> 
> ```go
> // Compare returns an integer comparing the two byte slices,
> // lexicographically.
> // The result will be 0 if a == b, -1 if a < b, and +1 if a > b
> func Compare(a, b []byte) int {
> 	for i := 0; i < len(a) && i < len(b); i++ {
> 		switch {
> 		case a[i] > b[i]:
> 			return 1
> 		case a[i] < b[i]:
> 			return -1
> 		}
> 	}
> 	switch {
> 	case len(a) > len(b):
> 		return 1
> 	case len(a) < len(b):
> 		return -1
> 	}
> 	return 0
> }
> ```

作为这一节的结束，此程序通过使用两个 switch 语句对字节数组进行比较：

```go
// Compare 按字典顺序比较两个字节切片并返回一个整数。
// 若 a == b，则结果为零；若 a < b；则结果为 -1；若 a > b，则结果为 +1。
func Compare(a, b []byte) int {
	for i := 0; i < len(a) && i < len(b); i++ {
		switch {
		case a[i] > b[i]:
			return 1
		case a[i] < b[i]:
			return -1
		}
	}
	switch {
	case len(a) > len(b):
		return 1
	case len(a) < len(b):
		return -1
	}
	return 0
}
```
### Type switch

### 类型选择

> A switch can also be used to discover the dynamic type of an interface variable. Such a type switch 
> uses the syntax of a type assertion with the keyword type inside the parentheses. If the switch 
> declares a variable in the expression, the variable will have the corresponding type in each clause. 
> It's also idiomatic to reuse the name in such cases, in effect declaring a new variable with the 
> same name but a different type in each case.
> 
> ```go
> var t interface{}
> t = functionOfSomeType()
> switch t := t.(type) {
> default:
> 	fmt.Printf("unexpected type %T", t)       // %T prints whatever type t has
> case bool:
> 	fmt.Printf("boolean %t\n", t)             // t has type bool
> case int:
> 	fmt.Printf("integer %d\n", t)             // t has type int
> case *bool:
> 	fmt.Printf("pointer to boolean %t\n", *t) // t has type *bool
> case *int:
> 	fmt.Printf("pointer to integer %d\n", *t) // t has type *int
> }
> ```

switch 也可用于判断接口变量的动态类型。如 类型选择 通过圆括号中的关键字 `type` 使用类型断言语法。若 
switch 在表达式中声明了一个变量，那么该变量的每个子句中都将有该变量对应的类型。在这些 case 中重用一
个名字也是符合语义的，实际上是在每个 case 里声明了一个不同类型但同名的新变量。

```go
var t interface{}
t = functionOfSomeType()
switch t := t.(type) {
default:
	fmt.Printf("unexpected type %T", t)       // %T 输出 t 是什么类型
case bool:
	fmt.Printf("boolean %t\n", t)             // t 是 bool 类型
case int:
	fmt.Printf("integer %d\n", t)             // t 是 int 类型
case *bool:
	fmt.Printf("pointer to boolean %t\n", *t) // t 是 *bool 类型
case *int:
	fmt.Printf("pointer to integer %d\n", *t) // t 是 *int 类型
}
```
