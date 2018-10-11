## Names

## 命名

Names are as important in Go as in any other language. They even have semantic effect: the 
visibility of a name outside a package is determined by whether its first character is upper case. 
It's therefore worth spending a little time talking about naming conventions in Go programs.

正如命名在其它语言中的地位，它在 Go 中同样重要。有时它们甚至会影响语义： 例如，某个名称在包外是否可
见，就取决于其首个字符是否为大写字母。 因此有必要花点时间来讨论 Go 程序中的命名约定。

### Package names

### 包名

When a package is imported, the package name becomes an accessor for the contents. After

当一个包被导入后，包名就会成了内容的访问器。在

``` go
import "bytes"
```
the importing package can talk about bytes.Buffer. It's helpful if everyone using the package can 
use the same name to refer to its contents, which implies that the package name should be good: 
short, concise, evocative. By convention, packages are given lower case, single-word names; there 
should be no need for underscores or mixedCaps. Err on the side of brevity, since everyone using 
your package will be typing that name. And don't worry about collisions a priori. The package name 
is only the default name for imports; it need not be unique across all source code, and in the rare 
case of a collision the importing package can choose a different name to use locally. In any case, 
confusion is rare because the file name in the import determines just which package is being used.

之后，被导入的包就能通过 bytes.Buffer 来引用了。 若所有人都能以相同的名称来引用其内容，这将大有裨
益，因此，包应当有个恰当的名称：其名称应该简洁明了而易于理解。按照惯例， 包应当以小写的单个单词来命
名，且不应使用下划线或驼峰记法。err 的命名就是出于简短考虑的，因为任何使用该包的人都会键入该名称。 
不必担心引用次序的冲突。包名就是导入时所需的唯一默认名称， 它并不需要在所有源码中保持唯一，即便在少
数发生冲突的情况下， 也可为导入的包选择一个别名来局部使用。 无论如何，通过文件名来判定使用的包，都是
不会产生混淆的。

Another convention is that the package name is the base name of its source directory; the package in 
src/encoding/base64 is imported as "encoding/base64" but has name base64, not encoding_base64 and 
not encodingBase64.

另一个约定就是包名应为其源码目录的基本名称。在 src/pkg/encoding/base64 中的包应作为 "encoding/
base64" 导入，其包名应为 base64， 而非 encoding_base64 或 encodingBase64。

The importer of a package will use the name to refer to its contents, so exported names in the 
package can use that fact to avoid stutter. (Don't use the import . notation, which can simplify 
tests that must run outside the package they are testing, but should otherwise be avoided.) For 
instance, the buffered reader type in the bufio package is called Reader, not BufReader, because 
users see it as bufio.Reader, which is a clear, concise name. Moreover, because imported entities 
are always addressed with their package name, bufio.Reader does not conflict with io.Reader. 
Similarly, the function to make new instances of ring.Ring—which is the definition of a constructor 
in Go—would normally be called NewRing, but since Ring is the only type exported by the package, and 
since the package is called ring, it's called just New, which clients of the package see as 
ring.New. Use the package structure to help you choose good names.

包的导入者可通过包名来引用其内容，因此包中的可导出名称可以此来避免冲突。 （请勿使用 import . 记法，
它可以简化必须在被测试包外运行的测试， 除此之外应尽量避免使用。）例如，bufio 包中的缓存读取器类型叫
做 Reader 而非 BufReader，因为用户将它看做 bufio.Reader，这是个清楚而简洁的名称。 此外，由于被导入的
项总是通过它们的包名来确定，因此 bufio.Reader 不会与 io.Reader 发生冲突。同样，用于创建 ring.Ring 的
新实例的函数（这就是 Go 中的构造函数）一般会称之为 NewRing，但由于 Ring 是该包所导出的唯一类型，且该
包也叫 ring，因此它可以只叫做 New，它跟在包的后面，就像 ring.New。使用包结构可以帮助你选择好的名称。

Another short example is once.Do; once.Do(setup) reads well and would not be improved by writing 
once.DoOrWaitUntilDone(setup). Long names don't automatically make things more readable. A helpful 
doc comment can often be more valuable than an extra long name.

另一个简短的例子是 once.Do，once.Do(setup) 表述足够清晰， 使用 once.DoOrWaitUntilDone(setup) 完全就
是画蛇添足。 长命名并不会使其更具可读性。一份有用的说明文档通常比额外的长名更有价值。

### Getters

### 获取器

Go doesn't provide automatic support for getters and setters. There's nothing wrong with providing 
getters and setters yourself, and it's often appropriate to do so, but it's neither idiomatic nor 
necessary to put Get into the getter's name. If you have a field called owner (lower case, 
unexported), the getter method should be called Owner (upper case, exported), not GetOwner. The use 
of upper-case names for export provides the hook to discriminate the field from the method. A setter 
function, if needed, will likely be called SetOwner. Both names read well in practice:

Go 并不对获取器（getter）和设置器（setter）提供自动支持。 你应当自己提供获取器和设置器，通常很值得这
样做，但若要将 Get 放到获取器的名字中，既不符合习惯，也没有必要。若你有个名为 owner （小写，未导出）
的字段，其获取器应当名为 Owner（大写，可导出）而非 GetOwner。大写字母即为可导出的这种规定为区分方法
和字段提供了便利。 若要提供设置器方法，SetOwner 是个不错的选择。两个命名看起来都很合理：

``` go
owner := obj.Owner()
if owner != user {
	obj.SetOwner(user)
}
```

### Interface names

### 接口名

By convention, one-method interfaces are named by the method name plus an -er suffix or similar 
modification to construct an agent noun: Reader, Writer, Formatter, CloseNotifier etc.

按照约定，只包含一个方法的接口应当以该方法的名称加上 - er 后缀来命名，如 Reader、Writer、 
Formatter、CloseNotifier 等。

There are a number of such names and it's productive to honor them and the function names they 
capture. Read, Write, Close, Flush, String and so on have canonical signatures and meanings. To 
avoid confusion, don't give your method one of those names unless it has the same signature and 
meaning. Conversely, if your type implements a method with the same meaning as a method on a well-
known type, give it the same name and signature; call your string-converter method String not 
ToString.

诸如此类的命名有很多，遵循它们及其代表的函数名会让事情变得简单。 Read、Write、Close、Flush、 String 
等都具有典型的签名和意义。为避免冲突，请不要用这些名称为你的方法命名， 除非你明确知道它们的签名和意
义相同。反之，若你的类型实现了的方法， 与一个众所周知的类型的方法拥有相同的含义，那就使用相同的命
名。 请将字符串转换方法命名为 String 而非 ToString。

### MixedCaps

### 驼峰记法

Finally, the convention in Go is to use MixedCaps or mixedCaps rather than underscores to write 
multiword names.

最后，Go 中约定使用驼峰记法 MixedCaps 或 mixedCaps 而非下划线的方式来对多单词名称进行命名。
