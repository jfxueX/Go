# Interfaces [I][1] & [II][2]

14 June 2017

[Golang tutorial series][3]

[1]: https://golangbot.com/interfaces-part-1/
[2]: https://golangbot.com/interfaces-part-2/
[3]: https://golangbot.com/learn-golang-series/

## Part I

### What is an interface?

The general definition of an interface in the Object oriented world is 
**"interface defines the behaviour of an object"**. It only specifies what the 
object is supposed to do. The way of achieving this behaviour (implementation 
detail) is upto the object.

In Go, an interface is a set of method signatures. When a type provides 
definition for all the methods in the interface, it is said to implement the 
interface. It is much similar to the OOP world. **Interface specifies what 
methods a type should have and the type decides how to implement these 
methods.**

For example *WashingMachine* can be a interface with method signatures 
*Cleaning()* and *Drying()*. Any type which provides definition for *Cleaning()* 
and *Drying()* is said to implement the *WashingMachine* interface.

### Declaring and implementing an interface

Lets dive right into a program which creates a interface and implements it.

```go
package main

import (  
    "fmt"
)

// interface definition
type VowelsFinder interface {  
    FindVowels() []rune
}

type MyString string

// MyString implements VowelsFinder
func (ms MyString) FindVowels() []rune {  
    var vowels []rune
    for _, rune := range ms {
        if rune == 'a' || rune == 'e' || rune == 'i' || rune == 'o' || rune == 'u' {
            vowels = append(vowels, rune)
        }
    }
    return vowels
}

func main() {  
    name := MyString("Sam Anderson")
    var v VowelsFinder
    v = name // possible since MyString implements VowelsFinder
    fmt.Printf("Vowels are %c", v.FindVowels())
}
```

[Run in playground](https://play.golang.org/p/F-T3S_wNNB)

Line no. 8 of the program above creates a interface type named `VowelsFinder` 
which has one method `FindVowels() []rune`.

In the next line a type `MyString` is created.

**In line no. 15 we add the method `FindVowels() []rune` to the receiver type 
`MyString`. Now `MyString` is said to implement the interface `VowelsFinder`. 
This is quite different from other languages like Java where a class has to 
explicitly state that it implements a interface using the `implements` keyword. 
This is not needed in go and go interfaces are implemented implicitly if a type 
contains all the methods declared in the interface.**

In line no.28, we assign `name` which is of type `MyString` to v of type 
`VowelsFinder`. This is possible since `MyString` implements `VowelsFinder`. 
`v.FindVowels()` in the next line calls the FindVowels method on `MyString` type 
and prints all the vowels in the string `Sam Anderson`. This program outputs 
`Vowels are [a e o]`

Congrats\! You have created and implemented your first interface.

### Practical use of interface

The above example taught us how to create and implement interfaces, but it 
didn't really show the practical use of a interface. Instead of `v.FindVowels()` 
if we used `name.FindVowels()` in the program above, it would have also worked 
and there would have been no use of the created interface.

So now lets look into a practical use case of interface.

We will write a simple program which calculates the total expense for a company 
based on the individual salaries of the employees. For brevity we have assumed 
that all expenses are in USD.

```go
package main

import (  
    "fmt"
)

type SalaryCalculator interface {  
    CalculateSalary() int
}

type Permanent struct {  
    empId    int
    basicpay int
    pf       int
}

type Contract struct {  
    empId  int
    basicpay int
}

// salary of permanent employee is sum of basic pay and pf
func (p Permanent) CalculateSalary() int {  
    return p.basicpay + p.pf
}

// salary of contract employee is the basic pay alone
func (c Contract) CalculateSalary() int {  
    return c.basicpay
}

/*
total expense is calculated by iterating though the SalaryCalculator slice and summing  
the salaries of the individual employees  
*/
func totalExpense(s []SalaryCalculator) {  
    expense := 0
    for _, v := range s {
        expense = expense + v.CalculateSalary()
    }
    fmt.Printf("Total Expense Per Month $%d", expense)
}

func main() {  
    pemp1 := Permanent{1, 5000, 20}
    pemp2 := Permanent{2, 6000, 30}
    cemp1 := Contract{3, 3000}
    employees := []SalaryCalculator{pemp1, pemp2, cemp1}
    totalExpense(employees)

}
```

[Run in playground](https://play.golang.org/p/5t6GgQ2TSU)

Line no. 7 of the above program declares `SalaryCalculator` interface type with 
a single method `CalculateSalary() int`.

We have two kind of employees in the company, `Permanent` and `Contract` defined 
by structs in line no. 11 and 17. The salary of permanent employees is the sum 
of the `basicpay` and `pf` whereas for contract employees it's just the basic 
pay `basicpay`. This is expressed in the corresponding `CalculateSalary` methods 
in line. no 23 and 28 respectively. By declaring this method, both `Permanent` 
and `Contract` now implement the `SalaryCalculator` interface.

The `totalExpense` function declared in line no.36 expresses the beauty of using 
interfaces. This method takes a slice of SalaryCalculator interface 
`[]SalaryCalculator` as parameter. In line no. 49 we pass a slice which contains 
both `Permanent` and `Contract` types to the `totalExpense` function. The 
`totalExpense` function calculates the expense by calling the `CalculateSalary` 
method of the corresponding type. This is done in line. no 39.

The biggest advantage of this is that `totalExpense` can be extended to any new 
employee type without any code changes. Lets say the company adds a new type of 
employee `Freelancer` with a different salary structure. This `Freelancer` can 
just be passed in the slice argument to `totalExpense` without even a single 
line of code change to the `totalExpense` function. This method will do what 
it's supposed to do as `Freelancer` will also implement the `SalaryCalculator` 
interface :).

This program outputs `Total Expense Per Month $14050`.


### Interface internal representation

An interface can be thought of as being represented internally by a tuple 
`(type, value)`. `type` is the underlying concrete type of the interface and 
`value` holds the value of the concrete type.

Lets write a program to understand better.

```go
package main

import (  
    "fmt"
)

type Tester interface {  
    Test()
}

type MyFloat float64

func (m MyFloat) Test() {  
    fmt.Println(m)
}

func describe(t Tester) {  
    fmt.Printf("Interface type %T value %v\n", t, t)
}

func main() {  
    var t Tester
    f := MyFloat(89.7)
    t = f
    describe(t)
    t.Test()
}
```

[Run in playground](https://play.golang.org/p/ZpQhhhs2920)

*Tester* interface has one method `Test()` and *MyFloat* type implements that 
interface. In line no. 24, we assign the variable `f` of type `MyFloat` to `t` 
which is of type `Tester`. Now the concrete type of t is `MyFloat` and the value 
of t is `89.7`. The `describe` function in line no.17 prints the value and 
concrete type of the interface. This program outputs

``` 
Interface type main.MyFloat value 89.7  
89.7  
```

### Empty Interface

An interface which has zero methods is called empty interface. It is represented 
as `interface{}`. Since the empty interface has zero methods, all types 
implement the empty interface.

```go
package main

import (  
    "fmt"
)

func describe(i interface{}) {  
    fmt.Printf("Type = %T, value = %v\n", i, i)
}

func main() {  
    s := "Hello World"
    describe(s)
    i := 55
    describe(i)
    strt := struct {
        name string
    }{
        name: "Naveen R",
    }
    describe(strt)
}
```

[Run in playground](https://play.golang.org/p/Fm5KescoJb)

In the program above, in line no.7, the `describe(i interface{})` function takes 
a empty interface as argument and hence it can be passed any type.

We pass string, int and struct to the `describe` function in line nos.  13, 15 
and 21 respectively. This program prints,

``` 
Type = string, value = Hello World  
Type = int, value = 55  
Type = struct { name string }, value = {Naveen R}  
```

### Type Assertion

Type assertion is used extract the underlying value of the interface.

**i.(T)** is the syntax which is used to get the underlying value of interface 
`i` whose concrete type is `T`.

A program is worth a thousand words 😀. Lets write one for type assertion.

```go
package main

import (  
    "fmt"
)

func assert(i interface{}) {  
    s := i.(int) //get the underlying int value from i
    fmt.Println(s)
}
func main() {  
    var s interface{} = 56
    assert(s)
}
```

[Run in playground](https://play.golang.org/p/YstKXEeSBL)

The concrete type of `s` in line no. 12 is `int`. We use the syntax `i.(int)` in 
line no. 8 to fetch the underlying int value of i. This program prints `56`.

What will happen if the concrete type in the above program is not int?  Well 
lets find out.

```go
package main

import (  
    "fmt"
)

func assert(i interface{}) {  
    s := i.(int) 
    fmt.Println(s)
}
func main() {  
    var s interface{} = "Steven Paul"
    assert(s)
}
```

[Run in playground](https://play.golang.org/p/88KflSceHK)

In the program above we pass `s` of concrete type `string` to the `assert` 
function which tries to extract a int value from it. This program will panic 
with the message `panic: interface conversion: interface {} is string, not int`.

To solve the above problem, we can use the syntax

```go
v, ok := i.(T)  
```

If the concrete type of `i` is `T` then `v` will have the underlying value of 
`i` and `ok` will be true.

If the concrete type of `i` is not `T` then `ok` will be false and `v` will have 
the zero value of type `T` and **the program will not panic**.

```go
package main

import (  
    "fmt"
)

func assert(i interface{}) {  
    v, ok := i.(int)
    fmt.Println(v, ok)
}
func main() {  
    var s interface{} = 56
    assert(s)
    var i interface{} = "Steven Paul"
    assert(i)
}
```

[Run in playground](https://play.golang.org/p/0sB-KlVw8A)

When `Steven Paul` is passed to the `assert` function, `ok` will be false since 
the concrete type of `i` is not `int` and `v` will have the value 0 which is the 
zero value of int. This program will print,

``` 
56 true  
0 false  
```

### Type Switch

A type switch is used to the compare the concrete type of an interface against 
multiple types specified in various case statements. It is similar to switch 
case. The only difference being the cases specify types and not values as in 
normal switch.

The syntax for type switch is similar to Type assertion. In the syntax `i.(T)` 
for Type assertion, the type `T` should be replaced by the keyword `type` for 
type switch. Lets see how this works in the program below.

```go
package main

import (  
    "fmt"
)

func findType(i interface{}) {  
    switch i.(type) {
    case string:
        fmt.Printf("I am a string and my value is %s\n", i.(string))
    case int:
        fmt.Printf("I am an int and my value is %d\n", i.(int))
    default:
        fmt.Printf("Unknown type\n")
    }
}
func main() {  
    findType("Naveen")
    findType(77)
    findType(89.98)
}
```

[Run in playground](https://play.golang.org/p/XYPDwOvoCh)

In line no. 8 of the above program, `switch i.(type)` specifies a type switch. 
Each of the case statements compare the concrete type of `i` to a specific type. 
If any case matches, the corresponding statement is printed. This program 
outputs,

``` 
I am a string and my value is Naveen  
I am an int and my value is 77  
Unknown type  
```

*89.98* in line no. 20 is of type `float64` and does not match any of the cases 
and hence `Unknown type` is printed in the last line.

**It is also possible to compare a type to an interface. If we have a type and 
if that type implements an interface, it is possible to compare this type with 
the interface it implements.**

Lets write a program for more clarity.

```go
package main

import "fmt"

type Describer interface {  
    Describe()
}
type Person struct {  
    name string
    age  int
}

func (p Person) Describe() {  
    fmt.Printf("%s is %d years old", p.name, p.age)
}

func findType(i interface{}) {  
    switch v := i.(type) {
    case Describer:
        v.Describe()
    default:
        fmt.Printf("unknown type\n")
    }
}

func main() {  
    findType("Naveen")
    p := Person{
        name: "Naveen R",
        age:  25,
    }
    findType(p)
}
```

[Run in background](https://play.golang.org/p/o6aHzIz4wC)

In the program above, the `Person` struct implements the `Describer` interface. 
In the case statement in line no. 19, `v` is compared to the `Describer` 
interface type. `p` implements `Describer` and hence this case is satisfied and 
`Describe()` method is called when the control hits `findType(p)` in line no. 
32.

This program outputs

``` 
unknown type  
Naveen R is 25 years old  
```

This brings us to the end of Interfaces Part I. We will continue our discussion 
about interfaces in Part II. Have a good day.

## Part II

### Implementing interfaces using pointer receivers vs value receivers

All the example interfaces we discussed in [part 1](#part-i) were implemented 
using value receivers. It is also possible to implement interfaces using pointer 
receivers. There is a subtlety to be noted while implementing interfaces using 
pointer receivers. Lets understand that using the following program.

```go
package main

import "fmt"

type Describer interface {  
    Describe()
}
type Person struct {  
    name string
    age  int
}

func (p Person) Describe() { //implemented using value receiver  
    fmt.Printf("%s is %d years old\n", p.name, p.age)
}

type Address struct {  
    state   string
    country string
}

func (a *Address) Describe() { //implemented using pointer receiver  
    fmt.Printf("State %s Country %s", a.state, a.country)
}

func main() {  
    var d1 Describer
    p1 := Person{"Sam", 25}
    d1 = p1
    d1.Describe()
    p2 := Person{"James", 32}
    d1 = &p2
    d1.Describe()

    var d2 Describer
    a := Address{"Washington", "USA"}

    /* compilation error if the following line is
       uncommented
       cannot use a (type Address) as type Describer
       in assignment: Address does not implement
       Describer (Describe method has pointer
       receiver)
    */
    //d2 = a

    d2 = &a //This works since Describer interface
    //is implemented by Address pointer in line 22
    d2.Describe()

}
```

[Run in playground](https://play.golang.org/p/IzspYiAQ82)

In the program above, the `Person` struct implements the `Describer` interface 
using value receiver in line no. 13.

As we have already learnt during our discussion about [methods][4], methods with 
value receivers accept both pointer and value receivers.  *It is legal to call a 
value method on anything which is a value or whose value can be dereferenced.*

*p1* is a value of type `Person` and it is assigned to `d1` in line no.  29. 
`Person` implements the `d1` interface and hence line no. 30 will print `Sam is 
25 years old`.

Similarly `d1` is assigned to `&p2` in line no. 32 and hence line no. 33 will 
print `James is 32 years old`. Awesome :).

The `Address` struct implements the `Describer` interface using pointer receiver 
in line no. 22.

If line. no 45 of the program above is uncommented, we will get the compilation 
error **main.go:42: cannot use a (type Address) as type Describer in assignment: 
Address does not implement Describer (Describe method has pointer receiver)**. 
This is because, the `Describer` interface is implemented using a Address 
Pointer receiver in line 22 and we are trying to assign `a` which is a value 
type and it has not implemented the `Describer` interface. This will definitely 
surprise you since we learnt earlier that [methods][5] with pointer receivers 
will accept both pointer and value receivers.  Then why isn't the code in line 
no. 45 working.

[4]: https://golangbot.com/methods#valuereceiversinmethodsvsvalueargumentsinfunctions
[5]: https://golangbot.com/methods/#pointerreceiversinmethodsvspointerargumentsinfunctions

**The reason is that it is legal to call a pointer-valued method on anything 
that is already a pointer or whose address can be taken. The concrete value 
stored in an interface is not addressable and hence it is not possible for the 
compiler to automatically take the address of `a` in line no. 45 and hence this 
code fails.**

Line no. 47 works because we are assigning the address of a `&a` to `d2`.

The rest of the program is self explanatory. This program will print,

``` 
Sam is 25 years old  
James is 32 years old  
State Washington Country USA  
```


### Implementing multiple interfaces

A type can implement more than one interface. Lets see how this is done in the 
following program.

```go
package main

import (  
    "fmt"
)

type SalaryCalculator interface {  
    DisplaySalary()
}

type LeaveCalculator interface {  
    CalculateLeavesLeft() int
}

type Employee struct {  
    firstName string
    lastName string
    basicPay int
    pf int
    totalLeaves int
    leavesTaken int
}

func (e Employee) DisplaySalary() {  
    fmt.Printf("%s %s has salary $%d", e.firstName, e.lastName, (e.basicPay + e.pf))
}

func (e Employee) CalculateLeavesLeft() int {  
    return e.totalLeaves - e.leavesTaken
}

func main() {  
    e := Employee {
        firstName: "Naveen",
        lastName: "Ramanathan",
        basicPay: 5000,
        pf: 200,
        totalLeaves: 30,
        leavesTaken: 5,
    }
    var s SalaryCalculator = e
    s.DisplaySalary()
    var l LeaveCalculator = e
    fmt.Println("\nLeaves left =", l.CalculateLeavesLeft())
}
```

[Run in playground](https://play.golang.org/p/DJxS5zxBcV)

The program above has two interfaces `SalaryCalculator` and `LeaveCalculator` 
declared in lines 7 and 11 respectively.

The `Employee` struct defined in line no. 15 provides implementations for the 
`DisplaySalary` method of `SalaryCalculator` interface in line no. 24 and the 
`CalculateLeavesLeft` method of `LeaveCalculator` interface interface in line 
no. 28. Now `Employee` implements both `SalaryCalculator` and `LeaveCalculator` 
interfaces.

In line no. 41 we assign `e` to a variable of type `SalaryCalculator` interface 
and in line no. 43 we assign the same variable `e` to a variable of type 
`LeaveCalculator`. This is possible since `e` which of type `Employee` 
implements both `SalaryCalculator` and `LeaveCalculator` interfaces.

This program outputs,

``` 
Naveen Ramanathan has salary $5200  
Leaves left = 25  
```

### Embedding interfaces

Although go does not offer inheritance, it is possible to create a new 
interfaces by embedding other interfaces.

Lets see how this is done.

```go
package main

import (  
    "fmt"
)

type SalaryCalculator interface {  
    DisplaySalary()
}

type LeaveCalculator interface {  
    CalculateLeavesLeft() int
}

type EmployeeOperations interface {  
    SalaryCalculator
    LeaveCalculator
}

type Employee struct {  
    firstName string
    lastName string
    basicPay int
    pf int
    totalLeaves int
    leavesTaken int
}

func (e Employee) DisplaySalary() {  
    fmt.Printf("%s %s has salary $%d", e.firstName, e.lastName, (e.basicPay + e.pf))
}

func (e Employee) CalculateLeavesLeft() int {  
    return e.totalLeaves - e.leavesTaken
}

func main() {  
    e := Employee {
        firstName: "Naveen",
        lastName: "Ramanathan",
        basicPay: 5000,
        pf: 200,
        totalLeaves: 30,
        leavesTaken: 5,
    }
    var empOp EmployeeOperations = e
    empOp.DisplaySalary()
    fmt.Println("\nLeaves left =", empOp.CalculateLeavesLeft())
}
```

[Run in playground](https://play.golang.org/p/Hia7D-WbZp)

*EmployeeOperations* interface in line 15 of the program above is created by 
embedding *SalaryCalculator* and *LeaveCalculator* interfaces.

Any type is said to implement `EmployeeOperations` interface if it provides 
method definitions for the methods present in both *SalaryCalculator* and 
*LeaveCalculator* interfaces.

The `Employee` struct implements `EmployeeOperations` interface since it 
provides definition for both `DisplaySalary` and `CalculateLeavesLeft` methods 
in lines 29 and 33 respectively.

In line 46, `e` of type `Employee` is assigned to `empOp` of type 
`EmployeeOperations`. In the next two lines, the `DisplaySalary()` and 
`CalculateLeavesLeft()` methods are called on `empOp`.

This program will output

``` 
Naveen Ramanathan has salary $5200  
Leaves left = 25  
```

### Zero value of Interface

The zero value of a interface is nil. A nil interface has both its underlying 
value and as well as concrete type as nil.

```go
package main

import "fmt"

type Describer interface {  
    Describe()
}

func main() {  
    var d1 Describer
    if d1 == nil {
        fmt.Printf("d1 is nil and has type %T value %v\n", d1, d1)
    }
}
```

[Run in playground](https://play.golang.org/p/vwYHC6Y78H)

*d1* in the above program is `nil` and this program will output

``` 
d1 is nil and has type <nil> value <nil>  
```

If we try to call a method on the `nil` interface, the program will panic since 
the `nil` interface neither has a underlying value nor a concrete type.

```go
package main

type Describer interface {  
    Describe()
}

func main() {  
    var d1 Describer
    d1.Describe()
}
```

[Run in playground](https://play.golang.org/p/rM-rY0uGTI)

Since `d1` in the program above is `nil`, this program will panic with runtime 
error **panic: runtime error: invalid memory address or nil pointer dereference  
\[signal SIGSEGV: segmentation violation code=0xffffffff addr=0x0 
pc=0xc8527\]"**

Thats it for interfaces. Have a good day.
