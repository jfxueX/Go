[Part 22: Channels][1]

23 July 2017

[Golang tutorial series][2].

[1]: https://golangbot.com/channels/
[2]: https://golangbot.com/learn-golang-series/
[3]: ../goroutines/gb-goroutines.md


* [What are channels](#what-are-channels)
* [Declaring channels](#declaring-channels)
* [Sending and receiving from channel](#sending-and-receiving-from-channel)
* [Sends and receives are blocking by default](#sends-and-receives-are-blocking-by-default)
* [Channel example program](#channel-example-program)
* [Another example for channels](#another-example-for-channels)
* [Deadlock](#deadlock)
* [Unidirectional channels](#unidirectional-channels)
* [Closing channels and for range loops on channels](#closing-channels-and-for-range-loops-on-channels)


In the [previous tutorial][3], we discussed about how concurrency is achieved in 
Go using Goroutines. In this tutorial we will discuss about channels and how 
Goroutines communicate using channels.

### What are channels

Channels can be thought as pipes using which Goroutines communicate.  Similar to 
how water flows from one end to another in a pipe, data can be sent from one end 
and received from the another end using channels.

### Declaring channels

Each channel has a type associated with it. This type is the type of data that 
the channel is allowed to transport. No other type is allowed to be transported 
using the channel.

*chan T* is a channel of type `T`

The zero value of a channel is `nil`. `nil` channels are not of any use and 
hence the channel has to be defined using `make` similar to [maps][4] and 
[slices][5].

[4]: https://golangbot.com/maps/
[5]: https://golangbot.com/arrays-and-slices/

Lets write some code that declares a channel.

```go
package main

import "fmt"

func main() {  
    var a chan int
    if a == nil {
        fmt.Println("channel a is nil, going to define it")
        a = make(chan int)
        fmt.Printf("Type of a is %T", a)
    }
}
```

[Run program in playground](https://play.golang.org/p/QDtf6mvymD)

The channel `a` declared in line no. 6 is `nil` as the zero value of a channel 
is `nil`. Hence the statements inside the if condition are executed and the 
channel is defined. `a` in the above program is a int channel. This program will 
output,

``` 
channel a is nil, going to define it  
Type of a is chan int  
```

As usual the short hand declaration is also a valid and concise way to define a 
channel.

```go
a := make(chan int)  
```

The above line of code also defines a int channel `a`.

### Sending and receiving from channel

The syntax to send and receive data from a channel are given below,

```go
data := <- a    // read from channel a  
a <- data       // write to channel a  
```

The direction of the arrow with respect to the channel specifies whether the 
data is sent or received.

In the first line, the arrow points outwards from `a` and hence we are reading 
from channel `a` and storing the value to the variable `data`.

In the second line, the arrow points towards `a` and hence we are writing to 
channel `a`.

### Sends and receives are blocking by default

Sends and receives to a channel are blocking by default. What does this mean? 
When a data is sent to a channel, the control is blocked in the send statement 
until some other Goroutine reads from that channel.  Similarly when data is read 
from a channel, the read is blocked until some Goroutine writes data to that 
channel.

This property of channels is what helps Goroutines communicate effectively 
without the use of explicit locks or conditional variables that are quite common 
in other programming languages.


### Channel example program

Enough of theory :). Lets write a program to understand how Goroutines 
communicate using channels.

We will actually rewrite the program we wrote when learning about [Goroutines][3] 
using channels here.

Let me quote the program here from the last tutorial.

```go
package main

import (  
    "fmt"
    "time"
)

func hello() {  
    fmt.Println("Hello world goroutine")
}

func main() {  
    go hello()
    time.Sleep(1 * time.Second)
    fmt.Println("main function")
}
```

[Run program in playground](https://play.golang.org/p/U9ZZuSql8-)

This was the program from the last tutorial. We use a sleep here to make the 
main Goroutine wait for the hello Goroutine to finish. If this doesn't make 
sense to you, I recommend reading the tutorial on [Goroutines][3]

We will rewrite the above program using channels.

```go
package main

import (  
    "fmt"
)

func hello(done chan bool) {  
    fmt.Println("Hello world goroutine")
    done <- true
}

func main() {  
    done := make(chan bool)
    go hello(done)
    <-done
    fmt.Println("main function")
}
```

[Run program in playground](https://play.golang.org/p/I8goKv6ZMF)

In the above program we create a `done` bool channel in line no. 12 and pass it 
as a parameter to the `hello` Goroutine. In line no. 14 we are receiving data 
from the `done` channel. This line of code is blocking which means that until 
some Goroutine writes data to the `done` channel, the control will not move to 
the next line of code. Hence this eliminates the need for the `time.Sleep` which 
was present in the original program to prevent the main Goroutine from exiting.

The line of code `<-done` receives data from the done Goroutine but does not use 
or store that data in any variable. This is perfectly legal.

Now we have our `main` Goroutine blocked waiting for data on done channel. The 
`hello` Goroutine receives this channel as parameter, prints `Hello world 
goroutine` and then writes to the `done` channel.  When this write is complete, 
the main Goroutine receives the data from the done channel, it is unblocked and 
then the text *main function* is printed.

This program outputs

``` 
Hello world goroutine  
main function  
```

Lets modify this program by introducing a sleep in the `hello` Goroutine to 
better understanding this blocking concept.

```go
package main

import (  
    "fmt"
    "time"
)

func hello(done chan bool) {  
    fmt.Println("hello go routine is going to sleep")
    time.Sleep(4 * time.Second)
    fmt.Println("hello go routine awake and going to write to done")
    done <- true
}

func main() {  
    done := make(chan bool)
    fmt.Println("Main going to call hello go goroutine")
    go hello(done)
    <-done
    fmt.Println("Main received data")
}
```

[Run in playground](https://play.golang.org/p/EejiO-yjUQ)

In the above program we have introduced a sleep of 4 seconds to the `hello` 
function in line no. 10.

This program will first print `Main going to call hello go goroutine`.  Then the 
`hello` Goroutine will be started and it will print `hello go routine is going to 
sleep`. After this is printed, the `hello` Goroutine will sleep for 4 seconds 
and during this time `main` Goroutine will be blocked since it is waiting for 
data from the done channel in line `<-done`. After 4 seconds `hello go routine 
awake and going to write to done` will be printed followed by `Main received 
data`.

### Another example for channels

Lets write one more program to understand channels better. This program will 
print the sum of the squares and cubes of the individual digits of a number.

For example if 123 is the input, then this program will calculate the output as

```go
squares = (1 * 1) + (2 * 2) + (3 * 3)  
cubes = (1 * 1 * 1) + (2 * 2 * 2) + (3 * 3 * 3)  
output = squares + cubes = 50
```

We will structure the program such that the squares are calculated in a separate 
Goroutine, cubes in another Goroutine and the final summation happens in the 
main Goroutine.

```go
package main

import (  
    "fmt"
)

func calcSquares(number int, squareop chan int) {  
    sum := 0
    for number != 0 {
        digit := number % 10
        sum += digit * digit
        number /= 10
    }
    squareop <- sum
}

func calcCubes(number int, cubeop chan int) {  
    sum := 0 
    for number != 0 {
        digit := number % 10
        sum += digit * digit * digit
        number /= 10
    }
    cubeop <- sum
} 

func main() {  
    number := 589
    sqrch := make(chan int)
    cubech := make(chan int)
    go calcSquares(number, sqrch)
    go calcCubes(number, cubech)
    squares, cubes := <-sqrch, <-cubech
    fmt.Println("Final output", squares + cubes)
}
```

[Run program in playground](https://play.golang.org/p/4RKr7_YO_B)


The `calcSquares` function in line no. 7 calculates the sum of the squares of 
the individual digits of the number and sends it to the `squareop` channel. 
Similarly the `calcCubes` function in line no. 17 calculates the sum of cubes of 
the individual digits of the number and sends it to the `cubeop` channel.

These two functions are run as separate Goroutines in line no. 31 and 32 and 
each is passed a channel to write to as the parameter. The main Goroutine waits 
for data from both these channels in line no. 33. Once the data is received from 
both the channels, they are stored in `squares` and `cubes` variables and the 
final output is computed and printed. This program will print

``` 
Final output 1536  
```

### Deadlock

One important factor to consider while using channels is deadlock. If a 
Goroutine is sending data on a channel, then it is expected that some other 
Goroutine should be receiving the data. If this does not happen, then the 
program will panic at runtime with `Deadlock`.

Similarly if a Goroutine is waiting to receive data from a channel, then some 
other Goroutine is expected to write data on that channel, else the program will 
panic.

```go
package main


func main() {  
    ch := make(chan int)
    ch <- 5
}
```

[Run program in playground](https://play.golang.org/p/q1O5sNx4aW)

In the program above, a channel `ch` is created and we send `5` to the channel 
in line `ch <- 5`. In this program no other Goroutine is receiving data from the 
channel `ch`. Hence this program will panic with the following runtime error.

```
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:  
main.main()  
    /tmp/sandbox249677995/main.go:6 +0x80
```

### Unidirectional channels

All the channels we discussed so far are bidirectional channels, that is data 
can be both sent and received on them. It is also possible to create 
unidirectional channels, that is channels that only send or receive data.

```go
package main

import "fmt"

func sendData(sendch chan<- int) {  
    sendch <- 10
}

func main() {  
    sendch := make(chan<- int)
    go sendData(sendch)
    fmt.Println(<-sendch)
}
```

[Run program in playground](https://play.golang.org/p/PRKHxM-iRK)

In the above program, we create send only channel `sendch` in line no.  10. 
`chan<- int` denotes a send only channel as the arrow is pointing to `chan`. We 
try to receive data from a send only channel in line no. 12.  This is not 
allowed and when the program is run, the compiler will complain stating,

*main.go:11: invalid operation: \<-sendch (receive from send-only type chan\<- 
int)*

**All is well but what is the point of writing to a send only channel if it 
cannot be read from\!**

**This is where channel conversion comes into use. It is possible to convert a 
bidirectional channel to a send only or receive only channel but not the vice 
versa.**

```go
package main

import "fmt"

func sendData(sendch chan<- int) {  
    sendch <- 10
}

func main() {  
    chnl := make(chan int)
    go sendData(chnl)
    fmt.Println(<-chnl)
}
```

[Run program in playground](https://play.golang.org/p/aqi_rJ1U8j)

In line no. 10 of the program above, a bidirectional channel `chnl` is created. 
It is passed as a parameter to the `sendData` Goroutine in line no. 11. The 
`sendData` function converts this channel to a send only channel in line no. 5 
in the parameter `sendch chan<- int`. So now the channel is send only inside the 
`sendData` Goroutine but it's bidirectional in the main Goroutine. This program 
will print `10` as the output.

### Closing channels and for range loops on channels

Senders have the ability to close the channel to notify receivers that no more 
data will be sent on the channel.

Receivers can use an additional variable while receiving data from the channel 
to check whether the channel has been closed.

```go
v, ok := <- ch  
```

In the above statement `ok` is true if the value was received by a successful 
send operation to a channel. If `ok` is false it means that we are reading from 
a closed channel. The value read from a closed channel will be the zero value of 
the channel's type. For example if the channel is an `int` channel, then the 
value received from a closed channel will be `0`.

```go
package main

import (  
    "fmt"
)

func producer(chnl chan int) {  
    for i := 0; i < 10; i++ {
        chnl <- i
    }
    close(chnl)
}

func main() {  
    ch := make(chan int)
    go producer(ch)
    for {
        v, ok := <-ch
        if ok == false {
            break
        }
        fmt.Println("Received ", v, ok)
    }
}
```

[Run program in playground](https://play.golang.org/p/XWmUKDA2Ri)

In the program above, the `producer` Goroutine writes 0 to 9 to the `chnl` 
channel and then closes the channel. The main function has an infinite `for` 
loop in line no.16 which checks whether the channel is closed using the variable 
`ok` in line no. 18. If `ok` is false it means that the channel is closed and 
hence the loop is broken. Else the received value and the value of `ok` is 
printed. This program prints,

``` 
Received  0 true  
Received  1 true  
Received  2 true  
Received  3 true  
Received  4 true  
Received  5 true  
Received  6 true  
Received  7 true  
Received  8 true  
Received  9 true  
```

The **for range** form of the for loop can be used to receive values from a 
channel until it is closed.

Lets rewrite the program above using a for range loop.

```go
package main

import (  
    "fmt"
)

func producer(chnl chan int) {  
    for i := 0; i < 10; i++ {
        chnl <- i
    }
    close(chnl)
}

func main() {  
    ch := make(chan int)
    go producer(ch)
    for v := range ch {
        fmt.Println("Received ",v)
    }
}
```

[Run program in playground](https://play.golang.org/p/JJ3Ida1r_6)

The `for range` loop in line no. 16 receives data from the `ch` channel until it 
is closed. Once `ch` is closed, the loop automatically exits.  This program 
outputs,

``` 
Received  0  
Received  1  
Received  2  
Received  3  
Received  4  
Received  5  
Received  6  
Received  7  
Received  8  
Received  9  
```

The program from [Another example for channels][6] section can be rewritten with 
more code reusability using for range loop.

[6]: #another-example-for-channels

If you take a closer look at the program you can notice that the code which 
finds the individual digits of a number is repeated in both `calcSquares` 
function and `calcCubes` function. We will move that code to its own function 
and call it concurrently.

```go
package main

import (  
    "fmt"
)

func digits(number int, dchnl chan int) {  
    for number != 0 {
        digit := number % 10
        dchnl <- digit
        number /= 10
    }
    close(dchnl)
}

func calcSquares(number int, squareop chan int) {  
    sum := 0
    dch := make(chan int)
    go digits(number, dch)
    for digit := range dch {
        sum += digit * digit
    }
    squareop <- sum
}

func calcCubes(number int, cubeop chan int) {  
    sum := 0
    dch := make(chan int)
    go digits(number, dch)
    for digit := range dch {
        sum += digit * digit * digit
    }
    cubeop <- sum
}

func main() {  
    number := 589
    sqrch := make(chan int)
    cubech := make(chan int)
    go calcSquares(number, sqrch)
    go calcCubes(number, cubech)
    squares, cubes := <-sqrch, <-cubech
    fmt.Println("Final output", squares+cubes)
}
```

[Run program in playground](https://play.golang.org/p/oL86W9Ui03)

The `digits` function in the program above now contains the logic for getting 
the individual digits from a number and it is called by both `calcSquares` and 
`calcCubes` functions concurrently. Once there are no more digits in the number, 
the channel is closed in line no. 13. The `calcSquares` and `calcCubes` 
Goroutines listen on their respective channels using a `for range` loop until it 
is closed. The rest of the program is the same. This program will also print

``` 
Final output 1536  
```

This brings us to the end of this tutorial. There are few more concepts in 
channels such as *buffered channels*, *worker pools* and *select*. We will 
discuss about them in separate tutorials of their own. Thanks for reading. Have 
a good day.
