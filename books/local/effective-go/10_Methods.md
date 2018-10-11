## Methods

## 方法

### Pointers vs. Values

### 指针 vs. 值

As we saw with ByteSize, methods can be defined for any named type (except a pointer or an 
interface); the receiver does not have to be a struct.

正如 ByteSize 那样，我们可以为任何已命名的类型（除了指针或接口）定义方法； 接收者可不必为结构体。

In the discussion of slices above, we wrote an Append function. We can define it as a method on 
slices instead. To do this, we first declare a named type to which we can bind the method, and then 
make the receiver for the method a value of that type.

在之前讨论切片时，我们编写了一个 Append 函数。 我们也可将其定义为切片的方法。为此，我们首先要声明一
个已命名的类型来绑定该方法， 然后使该方法的接收者成为该类型的值。

```go
type ByteSlice []byte

func (slice ByteSlice) Append(data []byte) []byte {
	// Body exactly the same as above
}
```
```go
type ByteSlice []byte

func (slice ByteSlice) Append(data []byte) []byte {
	// 主体和前面相同。
}
```
This still requires the method to return the updated slice. We can eliminate that clumsiness by 
redefining the method to take a pointer to a ByteSlice as its receiver, so the method can overwrite 
the caller's slice.

我们仍然需要该方法返回更新后的切片。为了消除这种不便，我们可通过重新定义该方法， 将一个指向 
ByteSlice 的指针作为该方法的接收者， 这样该方法就能重写调用者提供的切片了。

```go
func (p *ByteSlice) Append(data []byte) {
	slice := *p
	// Body as above, without the return.
	*p = slice
}
```
```go
func (p *ByteSlice) Append(data []byte) {
	slice := *p
	// 主体和前面相同，但没有 return。
	*p = slice
}
```
In fact, we can do even better. If we modify our function so it looks like a standard Write method, 
like this,

其实我们做得更好。若我们将函数修改为与标准 Write 类似的方法，就像这样，

```go
func (p *ByteSlice) Write(data []byte) (n int, err error) {
	slice := *p
	// Again as above.
	*p = slice
	return len(data), nil
}
```
```go
func (p *ByteSlice) Write(data []byte) (n int, err error) {
	slice := *p
	// 依旧和前面相同。
	*p = slice
	return len(data), nil
}
```
then the type `*ByteSlice` satisfies the standard interface io.Writer, which is handy. For instance, 
we can print into one.

那么类型 `*ByteSlice` 就满足了标准的 io.Writer 接口，这将非常实用。 例如，我们可以通过打印将内容写
入。

```go
	var b ByteSlice
	fmt.Fprintf(&b, "This hour has %d days\n", 7)
```
We pass the address of a ByteSlice because only `*ByteSlice` satisfies io.Writer. The rule about 
pointers vs. values for receivers is that value methods can be invoked on pointers and values, but 
pointer methods can only be invoked on pointers.

我们将 ByteSlice 的地址传入，因为只有 `*ByteSlice` 才满足 io.Writer。以指针或值为接收者的区别在于：
值方法可通过指针和值调用， 而指针方法只能通过指针来调用。

This rule arises because pointer methods can modify the receiver; invoking them on a value would 
cause the method to receive a copy of the value, so any modifications would be discarded. The 
language therefore disallows this mistake. There is a handy exception, though. When the value is 
addressable, the language takes care of the common case of invoking a pointer method on a value by 
inserting the address operator automatically. In our example, the variable b is addressable, so we 
can call its Write method with just b.Write. The compiler will rewrite that to (&b).Write for us.

之所以会有这条规则是因为指针方法可以修改接收者；通过值调用它们会导致方法接收到该值的副本， 因此任何
修改都将被丢弃，因此该语言不允许这种错误。不过有个方便的例外：若该值是可寻址的， 那么该语言就会自动
插入取址操作符来对付一般的通过值调用的指针方法。在我们的例子中，变量 b 是可寻址的，因此我们只需通过 
b.Write 来调用它的 Write 方法，编译器会将它重写为 (&b).Write。

By the way, the idea of using Write on a slice of bytes is central to the implementation of 
bytes.Buffer.

顺便一提，在字节切片上使用 Write 的想法已被 bytes.Buffer 所实现。
