[Part 16: Structures][1]
=======================

20 May 2017

[Golang tutorial series][2]

[1]: ://golangbot.com/structs/ 
[2]: https://golangbot.com/learn-golang-series/


* [What is a structure?](#what-is-a-structure)
* [Declaring a structure](#declaring-a-structure)
* [Creating named structures](#creating-named-structures)
* [Creating anonymous structures](#creating-anonymous-structures)
* [Zero value of a structure](#zero-value-of-a-structure)
* [Accessing individual fields of a struct](#accessing-individual-fields-of-a-struct)
* [Pointers to a struct](#pointers-to-a-struct)
* [Anonymous fields](#anonymous-fields)
* [Nested structs](#nested-structs)
* [Promoted fields](#promoted-fields)
* [Exported Structs and Fields](#exported-structs-and-fields)
* [Structs Equality](#structs-equality)

### What is a structure?

A structure is a user defined type which represents a collection of fields. It 
can be used in places where it makes sense to group the data into a single unit 
rather than maintaining each of them as separate types.

For instance a employee has a firstName, lastName and age. It makes sense to 
group these three properties into a single structure `employee`.

### Declaring a structure

```go
type Employee struct {  
    firstName string
    lastName  string
    age       int
}
```

The above snippet declares a structure type `Employee` which has fields 
firstName, lastName and age. This structure can also be made more compact by 
declaring fields that belong to a same type in a single line followed by the 
type name. In the above struct `firstName` and `lastName` belong to the same 
type `string` and hence the struct can be rewritten as

```go
type Employee struct {  
    firstName, lastName string
    age, salary         int
}
```

The above `Employee` struct is called a **named structure** because it creates a 
new type named `Employee` which can be used to create structures of type 
`Employee`.

It is possible to declare structures without declaring a new type and these type 
of structures are called **anonymous structures**.

```go
var employee struct {  
    firstName, lastName string
    age int
}
```

The above snippet creates a **anonymous structure** `employee`.

### Creating named structures

Lets define a **named structure Employee** using a simple program.

```go
package main

import (  
    "fmt"
)

type Employee struct {  
    firstName, lastName string
    age, salary         int
}

func main() {

    // creating structure using field names
    emp1 := Employee{
        firstName: "Sam",
        age:       25,
        salary:    500,
        lastName:  "Anderson",
    }

    // creating structure without using field names
    emp2 := Employee{"Thomas", "Paul", 29, 800}

    fmt.Println("Employee 1", emp1)
    fmt.Println("Employee 2", emp2)
}
```

[Run in playground](https://play.golang.org/p/uhPAHeUwvK)

In line no.7 of the above program, we create a named struct `Employee`.  In line 
no.15 of the above program, the `emp1` structure is defined by specifying the 
value for each field name. It is not necessary that the order of the field names 
should be same as that while declaring the structure type. Here we have changed 
the position of `lastName` and moved it to the last. This will work without any 
problems.

In line 23. of the above program, `emp2` is defined by omitting the field names. 
In this case it is necessary to maintain the order of fields to be the same as 
specified in the structure declaration.

The [program](https://play.golang.org/p/uhPAHeUwvK) above outputs.

```bash
Employee 1 {Sam Anderson 25 500}  
Employee 2 {Thomas Paul 29 800}  
```

### Creating anonymous structures

```go
package main

import (  
    "fmt"
)

func main() {  
    emp3 := struct {
        firstName, lastName string
        age, salary         int
    }{
        firstName: "Andreah",
        lastName:  "Nikola",
        age:       31,
        salary:    5000,
    }

    fmt.Println("Employee 3", emp3)
}
```

[Run in playground](https://play.golang.org/p/TEMFM3oZiq)

In line no 8. of the above program, an **anonymous structure variable** `emp3` 
is defined. As we have already mentioned, this structure is called anonymous 
because it only creates a new struct variable `emp3` and does not define any new 
struct type.

This program outputs,

```bash 
Employee 3 {Andreah Nikola 31 5000}  
```

### Zero value of a structure

When a struct is defined and it is not explicitly initialised with any value, 
the fields of the struct are assigned their zero values by default.

```go
package main

import (  
    "fmt"
)

type Employee struct {  
    firstName, lastName string
    age, salary         int
}

func main() {  
    var emp4 Employee // zero valued structure
    fmt.Println("Employee 4", emp4)
}
```

[Run in playground](https://play.golang.org/p/p7_OpVdFXJ)

The above program defines `emp4` but it is not initialised with any value. Hence 
`firstName` and `lastName` are assigned the zero values of string which is "" 
and `age`, `salary` are assigned the zero values of int which is 0. This program 
outputs

```bash
Employee 4 {  0 0}  
```

It is also possible to specify values for some fields and ignore the rest. In 
this case, the ignored field names are assigned zero values.

```go
package main

import (  
    "fmt"
)

type Employee struct {  
    firstName, lastName string
    age, salary         int
}

func main() {  
    emp5 := Employee{
        firstName: "John",
        lastName:  "Paul",
    }
    fmt.Println("Employee 5", emp5)
}
```

[Run in playground](https://play.golang.org/p/w2gPoCnlZ1)

In the above program in line. no 14 and 15, `firstName` and `lastName` are 
initialised whereas `age` and `salary` are not. Hence `age` and `salary` are 
assigned their zero values. This program outputs,

``` 
Employee 5 {John Paul 0 0}  
```

### Accessing individual fields of a struct

The dot `.` operator is used to access the individual fields of a structure.

```go
package main

import (  
    "fmt"
)

type Employee struct {  
    firstName, lastName string
    age, salary         int
}

func main() {  
    emp6 := Employee{"Sam", "Anderson", 55, 6000}
    fmt.Println("First Name:", emp6.firstName)
    fmt.Println("Last Name:", emp6.lastName)
    fmt.Println("Age:", emp6.age)
    fmt.Printf("Salary: $%d", emp6.salary)
}
```

[Run in playground](https://play.golang.org/p/GPd_sT85IS)

**emp6.firstName** in the above program accesses the `firstName` field of the 
`emp6` struct. This program outputs,

``` 
First Name: Sam  
Last Name: Anderson  
Age: 55  
Salary: $6000  
```

It is also possible to create a zero struct and then assign values to its fields 
later.

```go
package main

import (  
    "fmt"
)

type Employee struct {  
    firstName, lastName string
    age, salary         int
}

func main() {  
    var emp7 Employee
    emp7.firstName = "Jack"
    emp7.lastName = "Adams"
    fmt.Println("Employee 7:", emp7)
}
```

[Run in playground](https://play.golang.org/p/ZEOx10g7nN)

In the above program `emp7` is defined and then later `firstName` and `lastName` 
are assigned values. This program outputs,

``` 
Employee 7: {Jack Adams 0 0}  
```

### Pointers to a struct

It is also possible to create pointers to a struct.

```go
package main

import (  
    "fmt"
)

type Employee struct {  
    firstName, lastName string
    age, salary         int
}

func main() {  
    emp8 := &Employee{"Sam", "Anderson", 55, 6000}
    fmt.Println("First Name:", (*emp8).firstName)
    fmt.Println("Age:", (*emp8).age)
}
```

[Run in playground][3]

[3]: https://play.golang.org/p/xj87UCnBtH

**emp8** in the above program is a pointer to the `Employee` struct.  
`(*emp8).firstName` is the syntax to access the `firstName` field of the `emp8` 
struct. This [program][3] outputs,

``` 
First Name: Sam  
Age: 55  
```

**The language gives us the option to use `emp8.firstName` instead of the 
explicit dereference `(*emp8).firstName` to access the `firstName` field.**

```go
package main

import (  
    "fmt"
)

type Employee struct {  
    firstName, lastName string
    age, salary         int
}

func main() {  
    emp8 := &Employee{"Sam", "Anderson", 55, 6000}
    fmt.Println("First Name:", emp8.firstName)
    fmt.Println("Age:", emp8.age)
}
```

[Run in playground](https://play.golang.org/p/0ZE265qQ1h)

We have used `emp8.firstName` to access the `firstName` field in the above 
program and this program also outputs,

``` 
First Name: Sam  
Age: 55  
```

### Anonymous fields

It is possible to create structs with fields which contain only a type without 
the field name. These kind of fields are called anonymous fields.

The snippet below creates a struct `Person` which has two anonymous fields 
`string` and `int`

```go
type Person struct {  
    string
    int
}
```

Lets write a program using anonymous fields.

```go
package main

import (  
    "fmt"
)

type Person struct {  
    string
    int
}

func main() {  
    p := Person{"Naveen", 50}
    fmt.Println(p)
}
```

[Run in playground](https://play.golang.org/p/YF-DgdVSrC)

In the above program, `Person` is a struct with two anonymous fields. `p := 
Person{"Naveen", 50}` defines a variable of type `Person`. This program outputs 
`{Naveen 50}`.

**Even though an anonymous fields does not have a name, by default the name of a 
anonymous field is the name of its type.** For example in the case of the Person 
struct above, although the fields are anonymous, by default they take the name 
of the type of the fields. So `Person` struct has 2 fields with name `string` 
and `int`.

```go
package main

import (  
    "fmt"
)

type Person struct {  
    string
    int
}

func main() {  
    var p1 Person
    p1.string = "naveen"
    p1.int = 50
    fmt.Println(p1)
}
```

[Run in playground][4]

In line no. 14 and 15 of the above program, we access the anonymous fields of 
the Person struct using their types as field name which is "string" and "int" 
respectively. The output of the above [program][4]is ,

    {naveen 50}

[4]: https://play.golang.org/p/K-fGNxVyiA

### Nested structs

It is possible that a struct contains a field which in turn is a struct.  These 
kind of structs are called as nested structs.

```go
package main

import (  
    "fmt"
)

type Address struct {  
    city, state string
}
type Person struct {  
    name string
    age int
    address Address
}

func main() {  
    var p Person
    p.name = "Naveen"
    p.age = 50
    p.address = Address {
        city: "Chicago",
        state: "Illinois",
    }
    fmt.Println("Name:", p.name)
    fmt.Println("Age:", p.age)
    fmt.Println("City:", p.address.city)
    fmt.Println("State:", p.address.state)
}
```

[Run in playground](https://play.golang.org/p/46jkQFdTPO)

The `Person` struct in the above program has a field `address` which in turn is 
a struct. This program outputs

``` 
Name: Naveen  
Age: 50  
City: Chicago  
State: Illinois  
```

### Promoted fields

Fields that belong to a anonymous struct field in a structure are called 
promoted fields since they can be accessed as if they belong to the structure 
which holds the anonymous struct field. I can understand that this definition is 
quite complex so lets dive right into some code to understand this :).

```go
type Address struct {  
    city, state string
}
type Person struct {  
    name string
    age  int
    Address
}
```

In the above code snippet, the `Person` struct has an anonymous field `Address` 
which is a struct. Now the fields of the `Address` struct namely `city` and 
`state` are called promoted fields since they can be accessed as if they are 
directly declared in the `Person` struct itself.

```go
package main

import (
    "fmt"
)

type Address struct {
    city, state string
}
type Person struct {
    name string
    age  int
    Address
}

func main() {
    var p Person
    p.name = "Naveen"
    p.age = 50
    p.Address = Address{
        city:  "Chicago",
        state: "Illinois",
    }
    fmt.Println("Name:", p.name)
    fmt.Println("Age:", p.age)
    fmt.Println("City:", p.city)    // city is promoted field
    fmt.Println("State:", p.state)  // state is promoted field
}
```

[Run in playground](https://play.golang.org/p/OgeHCJYoEy)

In line no. 26 and 27 of the above program, the promoted fields `city` and 
`state` are accessed as if they are declared in the structure `p` itself using 
the syntax `p.city` and `p.state`. This program outputs,

``` 
Name: Naveen  
Age: 50  
City: Chicago  
State: Illinois  
```

### Exported Structs and Fields

If a struct type starts with a capital letter, then it is a exported type and it 
can be accessed from other packages. Similarly if the fields of a structure 
start with caps, they can be accessed from other packages.

Let's write a program which has custom packages to understand this better.

Create a folder named `structs` inside the `src` directory of your go workspace. 
Create another directory `computer` inside `structs`.

Inside the `computer` directory, save the program below with file name `spec.go`

```go
package computer

type Spec struct {  // exported struct
    Maker string    // exported field
    model string    // unexported field
    Price int       // exported field
}
```

The above snippet creates a package `computer` which contains a exported struct 
type `Spec` with two exported fields `Maker` and `Price` and one unexported 
field `model`. Lets import this package from the main package and use the `Spec` 
struct.

Create a file named `main.go` inside the `structs` directory and write the 
following program inside `main.go`

```go
package main

import "structs/computer"  
import "fmt"

func main() {  
    var spec computer.Spec
    spec.Maker = "apple"
    spec.Price = 50000
    fmt.Println("Spec:", spec)
}
```

The package structure should look like the following,

```
.
└── src
    └── structs
        ├── computer
        │   └── spec.go
        └── main.go
```

In line no.3 of the program above, we import the `computer` package. In line no. 
8 and 9, we access the two exported fields `Maker` and `Price` of the struct 
`Spec`. This program can be run by executing the commands `go install structs` 
followed by `workspacepath/bin/structs`

This program will output `Spec: {apple 50000}`.

If we try to access the unexported field `model`, the compiler will complain. 
Replace the contents of `main.go` with the following code.

```go
package main

import "structs/computer"  
import "fmt"

func main() {  
    var spec computer.Spec
    spec.Maker = "apple"
    spec.Price = 50000
    spec.model = "Mac Mini"
    fmt.Println("Spec:", spec)
}
```

In line no 10 of the above program, we try to access the unexported field 
`model`. Running this program will result in compilation error **spec.model 
undefined (cannot refer to unexported field or method model)**

### Structs Equality

**Structs are value types and are comparable if each of their fields are 
comparable. Two struct variables are considered equal if their corresponding 
fields are equal.**

```go
package main

import (
    "fmt"
)

type name struct {
    firstName string
    lastName string
}


func main() {
    name1 := name{"Steve", "Jobs"}
    name2 := name{"Steve", "Jobs"}
    if name1 == name2 {
        fmt.Println("name1 and name2 are equal")
    } else {
        fmt.Println("name1 and name2 are not equal")
    }

    name3 := name{firstName:"Steve", lastName:"Jobs"}
    name4 := name{}
    name4.firstName = "Steve"
    if name3 == name4 {
        fmt.Println("name3 and name4 are equal")
    } else {
        fmt.Println("name3 and name4 are not equal")
    }
}
```

[Run in playground](https://play.golang.org/p/AU1FkdsPk7)

In the above program, `name` struct type contain two string fields.  Since 
strings are comparable, it is possible to compare two struct variables of type 
`name`.

In the above program `name1` and `name2` are equal whereas `name3` and `name4` 
are not. This program will output,

``` 
name1 and name2 are equal  
name3 and name4 are not equal  
```

**Struct variables are not comparable if they contain fields which are not 
comparable** (Thanks to [alasijia][5] from reddit for pointing this out).

[5]: https://www.reddit.com/r/golang/comments/6cht1j/a_complete_guide_to_structs_in_go/dhvf7hd/

```go
package main

import (
    "fmt"
)

type image struct {
    data map[int]int
}

func main() {
    image1 := image{data: map[int]int{
        0: 155,
    }}
    image2 := image{data: map[int]int{
        0: 155,
    }}
    if image1 == image2 {
        fmt.Println("image1 and image2 are equal")
    }
}
```

[Run in playground](https://play.golang.org/p/T4svXOTYSg)

In the program above `image` struct type contains a field `data` which is of 
type `map`. maps are not comparable, hence `image1` and `image2` cannot be 
compared. If you run this program, compilation will fail with error 
**`main.go:18: invalid operation: image1 == image2 (struct containing 
map[int]int cannot be compared)`**.

The source code for this tutorial is available at
[github](https://github.com/golangbot/structs)

Thats it for structs in Go. Have a good day.
