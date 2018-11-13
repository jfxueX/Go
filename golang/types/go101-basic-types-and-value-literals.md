[Basic Types And Basic Value Literals][1]

[1]: https://go101.org/article/basic-types-and-value-literals.html

Types can be viewed as value templates, and values can be viewed as type 
instances. This article will introduce the built-in basic types and their value 
literals in Go. Composite types will not get introduced in this article.

### Built-in Basic Types In Go

Go supports following built-in basic types:

  - one boolean built-in boolean type: `bool`.
  - 11 built-in integer numeric types: `int8`, `uint8`, `int16`,
    `uint16`, `int32`, `uint32`, `int64`, `uint64`, `int`, `uint`, and
    `uintptr`.
  - two built-in floating-point numeric types: `float32` and `float64`.
  - two built-in complex numeric types: `complex64` and `complex128`.
  - one built-in string type: `string`.

Each of the 17 built-in basic types belongs to one different kind of type in Go. 
We can use the above built-in types in code without importing any packages, 
though all the names of these types are non-exported identifiers.

15 of the 17 built-in basic types are numeric types. Numeric types includes 
integer types, floating-point types and complex types.

Go also support two built-in alias types,

  - `byte` is a built-in alias of `uint8`. We can view `byte` and
    `uint8` as the same type.
  - `rune` is a built-in alias of `int32`. We can view `rune` and
    `int32` as the same type.

The integer types whose names starting with an `u` are unsigned types.  Values 
of unsigned types are always non-negative. The number in the name of a type 
means how many binary bits a value of the type will occupy in memory at run 
time. For example, every value of the `uint8` occupies 8 bits in memory. So the 
largest `uint8` value is `255` (2<sup>8</sup>-1), the largest `int8` value is 
`127` (2<sup>7</sup>-1), and the smallest `int8` value is `-128` (-2<sup>7</
sup>).

We often measure the size of a value based on the number of bytes it occupies in 
memory. One byte contains 8 bits. So the size of an `uint32` value is four 
bytes.

The sizes of `int` and `uint` values in memory are operation system dependent, 
either 32 or 64 bits. The size of `uintptr` value must be large enough to store 
the uninterpreted bits of any memory address.

The real and imaginary parts of a `complex64` value are both `float32` values, 
and the real and imaginary parts of a `complex128` value are both `float64` 
values.

In memory, all floating-point numeric values in Go are stored in [IEEE-754 
format][2].

A boolean value represents a truth. There are only two possible boolean values 
in memory, they are denoted by the two predeclared named constants, `false` and 
`true`. Name constants will be introduced in [the next article][3].

In logic, a string value denotes a piece of text. In memory, a string value 
stores a sequence of bytes, which is the UTF-8 encoding representation of the 
piece of text denoted by the string value. We can learn more facts on strings 
from the article [strings in Go][4] later.

[2]: https://en.wikipedia.org/wiki/IEEE_754
[3]: constants-and-variables.html
[4]: string.html
[5]: type-system-overview.html

Although there is only one built-in type for each of boolean and string types, 
we can define custom boolean and string types and declare alias types for the 
built-int boolean and string types. So there can be many boolean and string 
types. The same is for any kinds of numeric types.  The following are some type 
declarations examples. In these declarations, the word `type` is a keyword.

```go
// Some type definition declarations.
type status bool     // status and bool are two different types.
type MyString string // MyString and string are two different types.
type Id uint64       // Id and uint64 are two different types.
type real float32    // real and float32 are two different types.

// Some type alias declarations.
type boolean = bool // boolean and bool denote the same type.
type Text = string  // Text and string denote the same type.
type U8 = uint8     // U8, uint8 and byte denote the same type.
type char = rune    // char, rune and int32 denote the same type.
```

We can call the custom `real` type defined above and the built-in `float32` type 
both as float32 types. Note, the second ***float32*** word in the last sentence 
is a general reference, whereas the first one is a specified reference. 
Similarly, `MyString` and `string` are both string types, `status` and `bool` 
are both bool types, etc.

We can learn more on custom types in the article [overview of Go type system][5] 
later.

### Zero Values

Each type has a zero value. The zero value of a type can be viewed as the 
default value of the type.

  - The zero value of a boolean type a false truth.
  - The zero value of a numeric type is zero, though zeros of different numeric 
    types may have different sizes in memory.
  - The zero value of a string type is an empty string.

### Basic Value Literals

A literal of a value is a text representation of the value in code. A value may 
have many literals.

The literals denoting values of basic types are called basic value literal. A 
basic value literal is also called a literal constant, or an unnamed constant. 
Named constants will be introduced in the next article.

#### Boolean Value Literals

Go specification doesn't define boolean literals. However, in general 
programming, we can view the two predeclared identifiers, `false` and `true`, as 
boolean literals. But we should know that the two are not literals in the strict 
sense.

As above has mentioned, zero values of boolean types are denoted with the 
predeclared `false` constant.

#### Integer Value Literals

There are three integer value literal forms, the decimal (base 10) form, the 
octal (base 8) form and the hex (base 16) form. For example, the following three 
integer literals all denote `15` in decimal.

```go
0xF // the hex form (must start with a "0x" or "0X")
017 // the octal form (must start with a "0")
15  // the decimal form (mustn't start with a "0")
```

The following program will print two `true` texts.

```go
package main

func main() {
    println(15 == 017) // true
    println(15 == 0xF) // true
}
```

Note, the two `==` are the equal-to comparison operator, which will be 
introduced in [common operators][6].

Generally, zero values of integer types are denoted as `0` in literal, though 
there are many other legal literals for integer zero values, such as `00` and 
`0x0`. In fact, the zero value literals introduced in the current article for 
other kinds of numeric types can also represent the zero value of any integer 
type.

[6]: operators.html

#### Floating-Point Value Literals

A floating-point value literal can contain an integer part, a decimal point, a 
fractional part, and an exponent part. Some parts can be omitted. Example:

```go
1.23
01.23 // == 1.23
.23
1.
// A "e" or "E" starts the exponent part.
1.23e2  // == 123.0
123E2   // == 12300.0
123.E+2 // == 12300.0
1e-1    // == 0.1
.1e0    // == 0.1
0e+5    // == 0.0
```

The standard literals for zero value of floating-point types are `0.0`, though 
there are many other legal literals, such as `0.`, `.0`, `0e0`, etc. In fact, 
the zero value literals introduced in the current article for other kinds of 
numeric types can also represent the zero value of any floating-point type.

#### Imaginary Value Literals

An imaginary literal consists of a floating-point or decimal integer literal 
followed by the lower-case letter `i`. Examples:

    1.23i
    1.i
    .23i
    123i
    0123i   // == 123i
    1.23E2i // == 123i
    1e-1i

Imaginary literals are used to represent the imaginary parts of complex values. 
Here are some literals to denote complex values:

    1 + 2i       // == 1.0 + 2.0i
    1. - .1i     // == 1.0 + -0.1i
    1.23i - 7.89 // == -7.89 + 1.23i
    1.23i        // == 0.0 + 1.23i

The standard literals for zero values of complex types are `0.0+0.0i`, though 
there are many other legal literals, such as `0i`, `.0i`, `0+0i`, etc. In fact, 
the zero value literals introduced in the current article for other kinds of 
numeric types can also represent the zero value of any complex type.

#### Rune Value Literals

Rune types, including custom defined rune types and the built-in `rune` type 
(a.k.a., `int32` type), are special integer types, so all rune values can be 
denoted by the integer literals introduced above. On the other hand, many values 
of all kinds of integer types can also be represented by rune literals 
introduced below in the current subsection.

A rune value is intended to store a Unicode code point. Generally, we can view a 
break point as a Unicode character, but we should know that some Unicode 
characters are composed of more than one break points each.

A rune literal is expressed as one or more characters enclosed in a pair of 
quotes. The enclosed characters denote one Unicode break point value.  There are 
some minor variants of the rune literal form. The most popular form of rune 
literals is just to enclose the characters denoted by rune values between two 
single quotes. For example

    'a' // an English character
    'π'
    '众' // a Chinese character

The following rune literal variants are equivalent to `'a'` (the Unicode value 
of character `a` is 97).

    '\141'   // 141 is the octal representation of decimal number 97
    '\x61'   // 61 is the hex representation of decimal number 97
    '\u0061'
    '\U00000061'

Please note, <code>&#92;</code> must be followed by exactly three octal digits to represent a 
byte value, `\x` must be followed by exactly two hex digits to represent a byte 
value, `\u` must be followed by exactly four hex digits to represent a rune 
value, and `\U` must be followed by exactly eight hex digits to represent a rune 
value. Each such octal or hex digit sequence must represent a legal Unicode code 
point, otherwise it fails to compile.

The following program will print 7 `true` texts.

```go
package main

func main() {
    println('a' == 97)
    println('a' == '\141')
    println('a' == '\x61')
    println('a' == '\u0061')
    println('a' == '\U00000061')
    println(0x61 == '\x61')
    println('\u4f17' == '众')
}
```

In fact, the four variant rune literal forms just mentioned are rarely used for 
rune values in practice. They are occasionally used in interpreted string 
literals (see the next subsection for details).

If a rune literal is composed by two characters (not including the two quotes), 
the first one is the character <code>&#92;</code> and the second one is not a digital 
character, `x`, `u` and `U`, then the two successive characters will be escaped 
as one special character. The possible character pairs to be escaped are:

    \a   (Unicode value 0x07) alert or bell
    \b   (Unicode value 0x08) backspace
    \f   (Unicode value 0x0C) form feed
    \n   (Unicode value 0x0A) line feed or newline
    \r   (Unicode value 0x0D) carriage return
    \t   (Unicode value 0x09) horizontal tab
    \v   (Unicode value 0x0b) vertical tab
    \\   (Unicode value 0x5c) backslash
    \'   (Unicode value 0x27) single quote

`\n` is the most used escape character pair.

An example:

```go
    println('\n') // 10
    println('\r') // 13
    println('\'') // 39

    println('\n' == 10)     // true
    println('\n' == '\x0A') // true
```

There are many literals which can denote the zero values of rune types, such as 
`\000`, `\x00`, `\u0000`, etc. In fact, we can also use any numeric literal 
introduced above to represent the values of rune types, such as `0`, `0x0`, 
`0.0`, `0e0`, `0i`, etc.

#### String Value Literals

String values in Go are UTF-8 encoded. In fact, all Go source files must be 
UTF-8 encoding compatible.

There are two forms of string value literals, interpreted string literal (double 
quote form) and raw string literal (back quote form). For example, the following 
two string literals are equivalent:

```go
// The interpreted form.
"Hello\nworld!\n\"你好世界\""

// The raw form.
`Hello
world!
"你好世界"`
```

In the above interpreted string literal, each `\n` character pair will be 
escaped as one newline character, and each `\"` character pair will be escaped 
as one double quote character. Most of such escape character pairs are the same 
as the escape character pairs used in rune literals introduced above, except 
that `\"` is only legal in interpreted string literals and ``\` `` is only legal 
in rune literals.

The character sequence of <code>&#92;</code>, `\x`, `\u` and `\U` followed by several octal or 
hex digits introduced in the last section can also be used in interpreted string 
literals.

```go
// The following interpreted string literals are equivalent.
"\141\142\143"
"\x61\x62\x63"
"abc"

// The following interpreted string literals are equivalent.
"\u4f17\xe4\xba\xba" // The Unicode of 众 is 4f17, which is
                     // UTF-8 encoded as three bytes: e4 bc 97.
"\xe4\xbc\x97\u4eba" // The Unicode of 人 is 4eba, which is
                     // UTF-8 encoded as three bytes: e4 ba ba.
"\xe4\xbc\x97\xe4\xba\xba"
"众人"
```

Please note that each English character (code point) is represented with one 
byte, but each Chinese character (code point) is represented with three bytes.

In a raw string literal, no character sequences will be escaped. The back quote 
character is not allowed to appear in a raw string literal.  To get better 
cross-platform compatibility, carriage return characters (Unicode code point 
`0x0D`) inside raw string literals will be discarded.

Zero values of string types can be denoted as `""` or ` `` ` in literal.

### Representability Of Basic Value Literals

Any one string literal can represent values of any string types.

The predeclared `false` and `true` can represent values of any boolean types. 
But again, we should know that the two are not literals in the strict sense.

A numeric literal can be used to represent as an integer value only if it 
needn't to be rounded. For example, `1.23e2` can represent as values of any 
basic integer types, but `1.23` can't represent as values of any basic integer 
types. Rounding is allowed when using a numeric literal to represent as non- 
integer basic numeric values.

Each basic numeric type has a representable value range. So, if a literal 
overflows the value range of a type, then the literal is not representable as 
values of the type.

Some examples:

The Literal

The Types Which Values The Literal Can Represent (At Run Time)

`256`

All basic numeric types except int8 and uint8 types.

`255`

All basic numeric types except int8 types.

`-123`

All basic numeric types except the unsigned ones.

`123`

All basic numeric types.

`123.000`

`1.23e2`

`'a'`

`1.0+0i`

`1.23`

All basic floating-point and complex numeric types.

`0x10000000000000000`  
(16 zeros)

`3.5e38`

All basic floating-point and complex numeric types except float32 and
complex64 types.

`1+2i`

All basic complex numeric types.

`2e+308`

None basic types.

**Notes**

  - Because no values of the basic integer types provided in Go can hold 
    `0x10000000000000000`, so the literal is not representable as values of any 
    basic integer types.

  - The maximum IEEE-754 float32 value which can be represented accurately is 
    `3.40282346638528859811704183484516925440e+38`, so `3.5e38` is not 
    representable as values of any float32 and complex64 types.

  - The max IEEE-754 float64 value which can be represented accurately is 
    `1.797693134862315708145274237317043567981e+308`, so `2e+308` is not 
    representable as values of any `float64` and `complex128` types.

  - In the end, please note, although `0x10000000000000000` can represent values 
    of float32 types, however it can't represent any float32 values accurately in 
    memory. In other words, it will be rounded to the closest float32 value which 
    can be represented accurately in memory when it is used as values of float32 
    types.

-----

The ***Go 101*** project is hosted on both [Github][20] and [Gitlab][21]). 
Welcome to improve ***Go 101*** articles by submitting corrections for all kinds 
of mistakes, such as typos, grammar errors, wording inaccuracies, description 
flaws, code bugs and broken links.

Support Go 101 by playing [Tapir's games][22].

[20]: https://github.com/go101/go101
[21]: https://gitlab.com/Go101/go101
[22]: https://www.tapirgames.com
