# [Arrays and Slices][1]

20 April 2017

[Golang tutorial series][2]

[1]: https://golangbot.com/arrays-and-slices/
[2]: https://golangbot.com/learn-golang-series/


* [Arrays](#arrays)
    * [Declaration](#declaration)
    * [Arrays are value types](#arrays-are-value-types)
    * [Length of an array](#length-of-an-array)
    * [Iterating arrays using range](#iterating-arrays-using-range)
    * [Multidimensional arrays](#multidimensional-arrays)
* [Slices](#slices)
    * [Creating a slice](#creating-a-slice)
    * [modifying a slice](#modifying-a-slice)
    * [length and capacity of a slice](#length-and-capacity-of-a-slice)
    * [creating a slice using make](#creating-a-slice-using-make)
    * [Appending to a slice](#appending-to-a-slice)
    * [Passing a slice to a function](#passing-a-slice-to-a-function)
    * [Multidimensional slices](#multidimensional-slices)
    * [Memory Optimisation](#memory-optimisation)


### Arrays

An array is a collection of elements that belong to the same type. For example 
the collection of integers 5, 8, 9, 79, 76 form an array. Mixing values of 
different types, for example an array that contains both strings and integers is 
not allowed in Go.

#### Declaration

An array belongs to type `n[T]`. `n` denotes the number of elements in an array 
and `T` represents the type of each element. The number of elements `n` is also 
a part of the type(We will discuss this in more detail shortly.)

There are different ways to declare arrays. Lets look at them one by one.

```go
package main

import (  
    "fmt"
)


func main() {  
    var a [3]int    // int array with length 3
    fmt.Println(a)
}
```

[Run in playground](https://play.golang.org/p/Zvgh82u0ej)

**`var a [3]int`** declares a integer array of length 3. **All elements in an 
array are automatically assigned the zero value of the array type**. In this 
case `a` is an integer array and hence all elements of `a` are assigned to `0`, 
the zero value of int. Running the above program will **output** `[0 0 0]`.

The index of an array starts from `0` and ends at `length - 1`. Lets assign some 
values to the above array.

```go
package main

import (  
    "fmt"
)


func main() {  
    var a [3]int    // int array with length 3
    a[0] = 12       // array index starts at 0
    a[1] = 78
    a[2] = 50
    fmt.Println(a)
}
```

[Run in playground](https://play.golang.org/p/WF0Uj8sv39)

*`a[0]`* assigns value to the first element of the array. The program will 
**output** `[12 78 50]`

Lets create the same array using the **short hand declaration**.

```go
package main 

import (  
    "fmt"
)

func main() {  
    a := [3]int{12, 78, 50} // short hand declaration to create array
    fmt.Println(a)
}
```

[Run in playground](https://play.golang.org/p/NKOV04zgI6)

The program above will print the same **output** `[12 78 50]`

It is not necessary that all elements in an array have to be assigned a value 
during short hand declaration.

```go
package main

import (  
    "fmt"
)

func main() {  
    a := [3]int{12} 
    fmt.Println(a)
}
```

[Run in playground](https://play.golang.org/p/AdPH0kXRly)

In the above program in line no. 8 `a := [3]int{12}` declares an array of length 
3 but is provided with only one value `12`. The remaining 2 elements are 
assigned `0` automatically. The program will **output** `[12 0 0]`

You can even ignore the length of the array in the declaration and replace it 
with `...` and let the compiler find the length for you. This is done in the 
following program.

```go
package main

import (  
    "fmt"
)

func main() {  
    a := [...]int{12, 78, 50} // ... makes the compiler determine the length
    fmt.Println(a)
}
```

[Run in playground](https://play.golang.org/p/_fVmr6KGDh)

**The size of the array is a part of the type.** Hence `[5]int` and `[25]int` 
are distinct types. Because of this, arrays cannot be resized.  Don't worry 
about this restriction since `slices` exist to overcome this.

```go
package main

func main() {  
    a := [3]int{5, 78, 8}
    var b [5]int
    b = a   // not possible since [3]int and [5]int are distinct types
}
```

[Run in playground](https://play.golang.org/p/kBdot3pXSB)

In line no. 6 of the program above, we are trying to assign a variable of type 
`[3]int` to a variable of type `[5]int` which is not allowed and hence the 
compiler will throw error *`main.go:6: cannot use a (type [3]int) as type 
[5]int in assignment`*.

#### Arrays are value types

Arrays in Go are value types and not reference types. This means that when they 
are assigned to a new variable, a copy of the original array is assigned to the 
new variable. If changes are made to the new variable, it will not be reflected 
in the original array.

```go
package main

import "fmt"

func main() {  
    a := [...]string{"USA", "China", "India", "Germany", "France"}
    b := a  // a copy of a is assigned to b
    b[0] = "Singapore"
    fmt.Println("a is ", a)
    fmt.Println("b is ", b) 
}
```

[Run in playground](https://play.golang.org/p/-ncGk1mqPd)

In the above program in line no. 7, a copy of `a` is assigned to `b`. In line 
no. 8, the first element of `b` is changed to `Singapore`. This will not reflect 
in the original array `a`. The program will **output**,

``` 
a is [USA China India Germany France]  
b is [Singapore China India Germany France]  
```

Similarly when arrays are passed to functions as parameters, they are passed by 
value and the original array in unchanged.

```go
package main

import "fmt"

func changeLocal(num [5]int) {  
    num[0] = 55
    fmt.Println("inside function ", num)

}
func main() {  
    num := [...]int{5, 6, 7, 8, 8}
    fmt.Println("before passing to function ", num)
    changeLocal(num) //num is passed by value
    fmt.Println("after passing to function ", num)
}
```

[Run in playground](https://play.golang.org/p/e3U75Q8eUZ)

In the above program in line no. 13, the array `num` is actually passed by value 
to the function `changeLocal` and hence will not change because of the function 
call. This program will *output*,

``` 
before passing to function  [5 6 7 8 8]  
inside function  [55 6 7 8 8]  
after passing to function  [5 6 7 8 8]  
```

#### Length of an array

The length of the array is found by passing the array as parameter to the `len` 
function.

```go
package main

import "fmt"

func main() {  
    a := [...]float64{67.7, 89.8, 21, 78}
    fmt.Println("length of a is", len(a))

}
```

[Run in playground](https://play.golang.org/p/UrIeNlS0RN)

The **output** of the above program is `length of a is 4`

#### Iterating arrays using range

The `for` loop can be used to iterate over elements of an array.

```go
package main

import "fmt"

func main() {  
    a := [...]float64{67.7, 89.8, 21, 78}
    for i := 0; i < len(a); i++ {   // looping from 0 to the length of the array
        fmt.Printf("%d th element of a is %.2f\n", i, a[i])
    }
}
```

[Run in playground](https://play.golang.org/p/80ejSTACO6)

The above program uses a `for` loop to iterate over the elements of the array 
starting from index `0` to `length of the array - 1`. This program works and 
will print,

``` 
0 th element of a is 67.70  
1 th element of a is 89.80  
2 th element of a is 21.00  
3 th element of a is 78.00  
```

Go provides a better and concise way to iterate over an array by using the 
**range** form of the `for` loop. `range` returns both the index and the value 
at that index. Let's rewrite the above code using range. We will also find the 
sum of all elements of the array.

```go
package main

import "fmt"

func main() {  
    a := [...]float64{67.7, 89.8, 21, 78}
    sum := float64(0)
    for i, v := range a {   // range returns both the index and value
        fmt.Printf("%d the element of a is %.2f\n", i, v)
        sum += v
    }
    fmt.Println("\nsum of all elements of a",sum)
}
```

[Run in playground](https://play.golang.org/p/Ji6FRon36m)

line no. 8 `for i, v := range a` of the above program is the range form of the 
for loop. It will return both the index and the value at that index. We print 
the values and also calculate the sum of all elements of the array `a`. The 
**output** of the program is,

``` 
0 the element of a is 67.70  
1 the element of a is 89.80  
2 the element of a is 21.00  
3 the element of a is 78.00

sum of all elements of a 256.5  
```

In case you want only the value and want to ignore the index, you can do this by 
replacing the index with the `_` blank identifier.

```go
for _, v := range a { // ignores index  
}
```

The above for loop ignores the index. Similarly the value can also be ignored.

#### Multidimensional arrays

The arrays we created so far are all single dimension. It is possible to create 
multidimensional arrays.

```go
package main

import (  
    "fmt"
)

func printarray(a [3][2]string) {  
    for _, v1 := range a {
        for _, v2 := range v1 {
            fmt.Printf("%s ", v2)
        }
        fmt.Printf("\n")
    }
}

func main() {  
    a := [3][2]string{
        {"lion", "tiger"},
        {"cat", "dog"},
        {"pigeon", "peacock"},  // this comma is necessary. The compiler will complain if you omit this comma
    }
    printarray(a)
    var b [3][2]string
    b[0][0] = "apple"
    b[0][1] = "samsung"
    b[1][0] = "microsoft"
    b[1][1] = "google"
    b[2][0] = "AT&T"
    b[2][1] = "T-Mobile"
    fmt.Printf("\n")
    printarray(b)
}
```

[Run in playground](https://play.golang.org/p/InchXI4yY8)

In the above program in line no. 17, a two dimensional string array `a` has been 
declared using short hand syntax. The comma at the end of line no. 20 is 
necessary. This is because of the fact that the lexer automatically inserts 
semicolons according to simple rules. Please read 
<https://golang.org/doc/effective_go.html#semicolons> 
if you are interested to know more as to why this is needed.

Another 2d array `b` is declared in line no. 23 and strings are added to it one 
by one for each index. This is another way of initialising a 2d array.

The `printarray` function in line no. 7 uses two for range loops to print the 
contents of 2d arrays. The **output** of the above program is

``` 
lion tiger  
cat dog  
pigeon peacock 

apple samsung  
microsoft google  
AT&T T-Mobile  
```

Thats it for arrays. Although arrays seem to be flexible enough, they come with 
the restriction that they are of fixed length. It is not possible to increase 
the length of an array. This is were **slices** come into picture. In fact in 
Go, slices are more common than conventional arrays.

### Slices

A slice is a convenient, flexible and powerful wrapper on top of an array. 
Slices do not own any data on their own. They are the just references to 
existing arrays.

#### Creating a slice

A slice with elements of type T is represented by `[]T`

```go
package main

import (  
    "fmt"
)

func main() {  
    a := [5]int{76, 77, 78, 79, 80}
    var b []int = a[1:4]    // creates a slice from a[1] to a[3]
    fmt.Println(b)
}
```

[Run in playground](https://play.golang.org/p/Za6w5eubBB)

The syntax `a[start:end]` creates a slice from array `a` starting from index 
`start` to index `end - 1`. So in line no. 9 of the above program `a[1:4]` 
creates a slice representation of the array `a` starting from indexes 1 through 
3. Hence the slice `b` has values `[77 78 79]`.

Lets look at another way to create a slice.

```go
package main

import (  
    "fmt"
)

func main() {  
    c := []int{6, 7, 8}     // creates and array and returns a slice reference
    fmt.Println(c)
}
```

[Run in playground](https://play.golang.org/p/_Z97MgXavA)

In the above program in line no. 9, `c := []int{6, 7, 8}` creates an array with 
3 integers and returns a slice reference which is stored in c.

#### modifying a slice

A slice does not own any data of its own. It is just a representation of the 
underlying array. Any modifications done to the slice will be reflected in the 
underlying array.

```go
package main

import (  
    "fmt"
)

func main() {  
    darr := [...]int{57, 89, 90, 82, 100, 78, 67, 69, 59}
    dslice := darr[2:5]
    fmt.Println("array before",darr)
    for i := range dslice {
        dslice[i]++
    }
    fmt.Println("array after",darr) 
}
```

[Run in playground](https://play.golang.org/p/6FinudNf1k%20)

In line number 9 of the above program, we create `dslice` from indexes 2, 3, 4 
of the array. The for loop increments the value in these indexes by one. When we 
print the array after the for loop, we can see that the changes to the slice are 
reflected in the array. The output of the program is

``` 
array before [57 89 90 82 100 78 67 69 59]  
array after [57 89 91 83 101 78 67 69 59]  
```

When a number of slices share the same underlying array, the changes that each 
one makes will be reflected in the array.

```go
package main

import (  
    "fmt"
)

func main() {  
    numa := [3]int{78, 79 ,80}
    nums1 := numa[:]    // creates a slice which contains all elements of the array
    nums2 := numa[:]
    fmt.Println("array before change 1", numa)
    nums1[0] = 100
    fmt.Println("array after modification to slice nums1", numa)
    nums2[1] = 101
    fmt.Println("array after modification to slice nums2", numa)
}
```

[Run in playground](https://play.golang.org/p/mdNi4cs854)

In line no. 9, in `numa[:]` the start and end values are missing. The default 
values for start and end are `0` and `len(numa)` respectively.  Both slices 
`nums1` and `nums2` share the same array. The output of the program is

``` 
array before change 1 [78 79 80]  
array after modification to slice nums1 [100 79 80]  
array after modification to slice nums2 [100 101 80]  
```

From the output it's clear that when slices share the same array, the 
modifications which each one makes are reflected in the array.

#### length and capacity of a slice

The length of the slice is the number of elements in the slice. **The capacity 
of the slice is the number of elements in the underlying array starting from the 
index from which the slice is created.**

Lets write some code to understand this better.

```go
package main

import (  
    "fmt"
)

func main() {  
    fruitarray := [...]string{"apple", "orange", "grape", "mango", "water melon", "pine apple", "chikoo"}
    fruitslice := fruitarray[1:3]
    fmt.Printf("length of slice %d capacity %d", len(fruitslice), cap(fruitslice)) // length of is 2 and capacity is 6
}
```

[Run in playground](https://play.golang.org/p/a1WOcdv827)

In the above program, `fruitslice` is created from indexes 1 and 2 of the 
`fruitarray`. Hence the length of `fruitslice` is 2.

The length of the `fruitarray` is 7. `fruiteslice` is created from index `1` of 
`fruitarray`. Hence the capacity of `fruitslice` is the no of elements in 
`fruitarray` starting from index `1` i.e from `orange` and that value is `6`. 
Hence the capacity of fruitslice is 6. The [program][3] outputs **length of 
slice 2 capacity 6**.

[3]: https://play.golang.org/p/a1WOcdv827

A slice can be re-sliced upto its capacity. Anything beyond that will cause the 
program to throw a run time error.

```go
package main

import (  
    "fmt"
)

func main() {  
    fruitarray := [...]string{"apple", "orange", "grape", "mango", "water melon", "pine apple", "chikoo"}
    fruitslice := fruitarray[1:3]
    fmt.Printf("length of slice %d capacity %d\n", len(fruitslice), cap(fruitslice)) // length of is 2 and capacity is 6
    fruitslice = fruitslice[:cap(fruitslice)] // re-slicing furitslice till its capacity
    fmt.Println("After re-slicing length is", len(fruitslice), "and capacity is", cap(fruitslice))
}
```

[Run in playground](https://play.golang.org/p/GcNzOOGicu)

In line no. 11 of the above program,`fruitslice` is re-sliced to its capacity. 
The above program outputs,

``` 
length of slice 2 capacity 6  
After re-slicing length is 6 and capacity is 6  
```

#### creating a slice using make

*`func make([]T, len, cap) []T`* can be used to create a slice by passing the 
type, length and capacity. The capacity parameter is optional and defaults to 
the length. The make function creates an array and returns a slice reference to 
it.

```go
package main

import (  
    "fmt"
)

func main() {  
    i := make([]int, 5, 5)
    fmt.Println(i)
}
```

[Run in playground](https://play.golang.org/p/M4OqxzerxN)

The values are zeroed by default when a slice is created using make. The above 
program will output `[0 0 0 0 0]`.

#### Appending to a slice

As we already know arrays are restricted to fixed length and their length cannot 
be increased. Slices are dynamic and new elements can be appended to the slice 
using `append` function. The definition of append function is `func append(s 
[]T, x ...T) []T`.

**x ...T** in the function definition means that the function accepts variable 
number of arguments for the parameter x. These type of functions are called 
[variadic functions][4].

[4]: https://golangbot.com/variadic-functions/

One question might be bothering you though. If slices are backed by arrays and 
arrays themselves are of fixed length then how come a slice is of dynamic 
length. Well what happens under the hoods is, when new elements are appended to 
the slice, a new array is created. The elements of the existing array are copied 
to this new array and a new slice reference for this new array is returned. The 
capacity of the new slice is now twice that of the old slice. Pretty cool 
right :). The following program will make things clear.

```go
package main

import (  
    "fmt"
)

func main() {  
    cars := []string{"Ferrari", "Honda", "Ford"}
    fmt.Println("cars:", cars, "has old length", len(cars), "and capacity", cap(cars)) // capacity of cars is 3
    cars = append(cars, "Toyota")
    fmt.Println("cars:", cars, "has new length", len(cars), "and capacity", cap(cars)) // capacity of cars is doubled to 6
}
```

[Run in playground](https://play.golang.org/p/VUSXCOs1CF)

In the above program, the capacity of `cars` is 3 initially. We append a new 
element to cars in line no. 10 and assign the slice returned by `append(cars, 
"Toyota")` to cars again. Now the capacity of cars is doubled and becomes 6. The 
output of the above program is

``` 
cars: [Ferrari Honda Ford] has old length 3 and capacity 3  
cars: [Ferrari Honda Ford Toyota] has new length 4 and capacity 6  
```

The zero value of a slice type is `nil`. A `nil` slice has length and capacity 
0. It is possible to append values to a `nil` slice using the append function.

```go
package main

import (  
    "fmt"
)

func main() {  
    var names []string // zero value of a slice is nil
    if names == nil {
        fmt.Println("slice is nil going to append")
        names = append(names, "John", "Sebastian", "Vinay")
        fmt.Println("names contents:", names)
    }
}
```

[Run in playground](https://play.golang.org/p/x_-4XAJHbM)

In the above program `names` is nil and we have appended 3 strings to `names`. 
The output of the program is

``` 
slice is nil going to append  
names contents: [John Sebastian Vinay]  
```

It is also possible to append one slice to another using the `...` operator. You 
can learn more about this operator in the [variadic functions][5] tutorial.

[5]: https://golangbot.com/variadic-functions/

```go
package main

import (  
    "fmt"
)

func main() {  
    veggies := []string{"potatoes", "tomatoes", "brinjal"}
    fruits := []string{"oranges", "apples"}
    food := append(veggies, fruits...)
    fmt.Println("food:", food)
}
```

[Run in playground](https://play.golang.org/p/UnHOH_u6HS)

In line no. 10 of the above program *food* is created by appending `fruits` to 
`veggies`. Output of the program is `food: [potatoes tomatoes brinjal oranges 
apples]`

#### Passing a slice to a function

Slices can be thought of as being represented internally by a structure type. 
This is how it looks,

```go
type slice struct {  
    Length        int
    Capacity      int
    ZerothElement *byte
}
```

A slice contains the length, capacity and a pointer to the zeroth element of the 
array. When a slice is passed to a function, even though it's passed by value, 
the pointer variable will refer to the same underlying array. Hence when a slice 
is passed to a function as parameter, changes made inside the function are 
visible outside the function too. Lets write a program to check this out.

```go
package main

import (  
    "fmt"
)

func subtactOne(numbers []int) {  
    for i := range numbers {
        numbers[i] -= 2
    }

}
func main() {  
    nos := []int{8, 7, 6}
    fmt.Println("slice before function call", nos)
    subtactOne(nos)                               // function modifies the slice
    fmt.Println("slice after function call", nos) // modifications are visible outside
}
```

[Run in playground](https://play.golang.org/p/IzqDihNifq%20)

The function call in line number 17 of the above program decrements each element 
of the slice by 2. When the slice is printed after the function call, these 
changes are visible. If you can recall, this is different from an array where 
the changes made to an array inside a function are not visible outside the 
function. Output of the above [program][6] is,

``` 
slice before function call [8 7 6]  
slice after function call [6 5 4]  
```

[6]: https://play.golang.org/p/bWUb6R-1bS%20

#### Multidimensional slices

Similar to arrays, slices can have multiple dimensions.

```go
package main

import (  
    "fmt"
)


func main() {  
     pls := [][]string {
            {"C", "C++"},
            {"JavaScript"},
            {"Go", "Rust"},
            }
    for _, v1 := range pls {
        for _, v2 := range v1 {
            fmt.Printf("%s ", v2)
        }
        fmt.Printf("\n")
    }
}
```

[Run in playground](https://play.golang.org/p/--p1AvNGwN)

The output of the program is,

``` 
C C++  
JavaScript  
Go Rust  
```

#### Memory Optimisation

Slices hold a reference to the underlying array. As long as the slice is in 
memory, the array cannot be garbage collected. This might be of concern when it 
comes to memory management. Lets assume that we have a very large array and we 
are interested in processing only a small part of it. Henceforth we create a 
slice from that array and start processing the slice. The important thing to be 
noted here is that the array will still be in memory since the slice references 
it.

One way to solve this problem is to use the [copy][7] function `func copy(dst, 
src []T) int` to make a copy of that slice. This way we can use the new slice 
and the original array can be garbage collected.

[7]: https://golang.org/pkg/builtin/#copy

```go
package main

import (  
    "fmt"
)

func countries() []string {  
    countries := []string{"USA", "Singapore", "Germany", "India", "Australia"}
    neededCountries := countries[:len(countries)-2]
    countriesCpy := make([]string, len(neededCountries))
    copy(countriesCpy, neededCountries) // copies neededCountries to countriesCpy
    return countriesCpy
}
func main() {  
    countriesNeeded := countries()
    fmt.Println(countriesNeeded)
}
```

[Run in playground](https://play.golang.org/p/35ayYBhcDE)

In line no. 9 of the above program, `neededCountries := 
countries[:len(countries)-2]` creates a slice of `countries` barring the last 2 
elements. Line no. 11 of the above program copies `neededCountries` to 
`countriesCpy` and also returns it from the function in the next line. Now 
`countries` array can be garbage collected since `neededCountries` is no longer 
referenced.

I have compiled all the concepts we discussed so far into a single program. You 
can download it from [github][8].

[8]: https://github.com/golangbot/arraysandslices

Thats it for arrays and slices. Thanks for reading. Please leave your valuable 
feedback and comments.
