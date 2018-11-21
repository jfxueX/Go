[Part 24: Select][1]
====================

10 August 2017

[Golang tutorial series][2]

[1]: https://golangbot.com/select/
[2]: https://golangbot.com/learn-golang-series/


* [What is *select*?](#what-is-select)
* [Example](#example)
* [Practical use of select](#practical-use-of-select)
* [Default case](#default-case)
* [Deadlock and default case](#deadlock-and-default-case)
* [Random selection](#random-selection)
* [Gotcha - Empty select](#gotcha---empty-select)


### What is *select*?

The `select` statement is used to choose from multiple send/receive channel 
operations. The select statement blocks until one of the send/receive operation 
is ready. If multiple operations are ready, one of them is chosen at random. The 
syntax is similar to `switch` except that each of the case statement will be a 
channel operation. Lets dive right into some code for better understanding.

### Example

```go
package main

import (  
    "fmt"
    "time"
)

func server1(ch chan string) {  
    time.Sleep(6 * time.Second)
    ch <- "from server1"
}

func server2(ch chan string) {  
    time.Sleep(3 * time.Second)
    ch <- "from server2"

}

func main() {  
    output1 := make(chan string)
    output2 := make(chan string)
    go server1(output1)
    go server2(output2)
    select {
    case s1 := <-output1:
        fmt.Println(s1)
    case s2 := <-output2:
        fmt.Println(s2)
    }
}
```

[Run in playground](https://play.golang.org/p/3_yaJSoSpG)

In the program above, the `server1` function in line no. 8 sleeps for 6 seconds 
then writes the text *from server1* to the channel `ch`. The `server2` function 
in line no. 12 sleeps for 3 seconds and then writes *from server2* to the 
channel `ch`.

The main function calls the go Goroutines `server1` and `server2` in line nos. 
20 and 21 respectively.

In line no. 22, the control reaches the `select` statement. The `select` 
statement blocks until one of its cases is ready. In our program above, the 
`server1` Goroutine writes to the `output1` channel after 6 seconds whereas the 
`server2` writes to the `output2` channel after 3 seconds.  So the select 
statement will block for 3 seconds and will wait for `server2` Goroutine to 
write to the `output2` channel. After 3 seconds, the program prints,

``` 
from server2  
```

and then will terminate.


### Practical use of select

The reason behind naming the functions in the above program as `server1` and 
`server2` is to illustrate the practical use of *select*.

Lets assume we have a mission critical application and we need to return the 
output to the user as quickly as possible. The database for this application is 
replicated and stored in different servers across the world. Assume that the 
functions `server1` and `server2` are in fact communicating with 2 such servers. 
The response time of each server is dependant on the load on each and the 
network delay. We send the request to both the servers and then wait on the 
corresponding channels for the response using the `select` statement. The server 
which responds first is chosen by the select and the other response is ignored. 
This way we can send the same request to multiple servers and return the 
quickest response to the user :).

### Default case

The default case in a `select` statement is executed when none of the other case 
is ready. This is generally used to prevent the select statement from blocking.

```go
package main

import (  
    "fmt"
    "time"
)

func process(ch chan string) {  
    time.Sleep(10500 * time.Millisecond)
    ch <- "process successful"
}

func main() {  
    ch := make(chan string)
    go process(ch)
    for {
        time.Sleep(1000 * time.Millisecond)
        select {
        case v := <-ch:
            fmt.Println("received value: ", v)
            return
        default:
            fmt.Println("no value received")
        }
    }

}
```

[Run in playground](https://play.golang.org/p/8xS5r9g1Uy)

In the program above, the `process` function in line no. 8 sleeps for 10500 
milliseconds (10.5 seconds) and then writes `process successful` to the `ch` 
channel. This function is called concurrently in line no. 15 of the program.

After calling the `process` Goroutine concurrently, an infinite for loop is 
started in the main Goroutine. The infinite loop sleeps for 1000 milliseconds (1 
second) during the start of each iteration and them performs a select operation. 
During the first 10500 milliseconds, the first case of the select statement 
namely `case v := <-ch:` will not be ready since the `process` Goroutine will 
write to the `ch` channel only after 10500 milliseconds. Hence the`default` case 
will be executed during this time and the program will print `no value received` 
10 times.

After 10.5 seconds, the `process` Goroutine writes `process successful` to `ch` 
in line no. 10. Now the first case of the select statement will be executed and 
the program will print `received value: process successful` and then it will 
terminate. This program will output,

``` 
no value received  
no value received  
no value received  
no value received  
no value received  
no value received  
no value received  
no value received  
no value received  
no value received  
received value:  process successful  
```


### Deadlock and default case

```go
package main

func main() {  
    ch := make(chan string)
    select {
    case <-ch:
    }
}
```

[Run in playground](https://play.golang.org/p/za0GZ4o7HH)

In the program above, we have created a channel `ch` in line no. 4. We try to 
read from this channel inside the select in line no. 6. The select statement 
will block forever since no other Goroutine is writing to this channel and hence 
will result in deadlock. This program will panic at runtime with the following 
message,

```
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan receive]:  
main.main()  
    /tmp/sandbox416567824/main.go:6 +0x80
```

If a default case is present, this deadlock will not happen since the default 
case will be executed when no other case is ready. The program above is 
rewritten with a default case below.

```go
package main

import "fmt"

func main() {  
    ch := make(chan string)
    select {
    case <-ch:
    default:
        fmt.Println("default case executed")
    }
}
```

[Run in playground](https://play.golang.org/p/Pxsh_KlFUw)

The above program will print,

``` 
default case executed  
```

Similarly the default case will be executed even if the select has only nil 
channels.

```go
ackage main

import "fmt"

func main() {  
    var ch chan string
    select {
    case v := <-ch:
        fmt.Println("received value", v)
    default:
        fmt.Println("default case executed")

    }
}
```

[Run in playground](https://play.golang.org/p/IKmGpN61m1)

In the program above `ch` is `nil` and we are trying to read from `ch` in the 
select in line no. 8. If the `default` case was not present , the `select` would 
have blocked forever and caused a deadlock. Since we have a default case inside 
the select, it will be executed and the program will print,

``` 
default case executed  
```

### Random selection

When multiple cases in a `select` statement are ready, one of them will be 
executed at random.

```go
package main

import (  
    "fmt"
    "time"
)

func server1(ch chan string) {  
    ch <- "from server1"
}
func server2(ch chan string) {  
    ch <- "from server2"

}
func main() {  
    output1 := make(chan string)
    output2 := make(chan string)
    go server1(output1)
    go server2(output2)
    time.Sleep(1 * time.Second)
    select {
    case s1 := <-output1:
        fmt.Println(s1)
    case s2 := <-output2:
        fmt.Println(s2)
    }
}
```

[Run in playground](https://play.golang.org/p/vJ6VhVl9YY)

In the program above, the `server1` and `server2` go routines are called in line 
no. 18 and 19 respectively. Then the main program sleeps for 1 second in line 
no. 20. When the control reaches the `select` statement in line no. 21, 
`server1` would have written `from server1` to the `output1` channel and 
`server2` would have written `from server2` to the `output2` channel and hence 
both the cases of the select statement are ready to be executed. If you run this 
program multiple times, the output will vary between `from server1` or `from 
server2` depending on which case is chosen in random.

Please run this program in your local system to get this randomness. If this 
program is run in the playground it will print the same output since the 
playground is deterministic.

### Gotcha - Empty select

```go
package main

func main() {  
    select {}
}
```

[Run in playground](https://play.golang.org/p/u8hErIxgxs)

What do you think will be the output of the program above?

We know that the select statement will block until one of its cases is executed. 
In this case the select statement doesn't have any cases and hence it will block 
forever resulting in a deadlock. This program will panic with the following 
output,

```
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [select (no cases)]:  
main.main()  
    /tmp/sandbox299546399/main.go:4 +0x20
```

This brings us to an end of this tutorial. Have a good day.
