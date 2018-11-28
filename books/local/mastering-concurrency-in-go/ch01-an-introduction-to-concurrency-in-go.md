# Chapter 1. An Introduction to Concurrency in Go


While Go is both a great general purpose and low-level systems language, one of 
its primary strengths is the built-in concurrency model and tools. Many other 
languages have third-party libraries (or extensions), but inherent concurrency 
is something unique to modern languages, and it is a core feature of Go's 
design.

Though there's no doubt that Go excels at concurrency—as we'll see in this book—
what it has that many other languages lack is a robust set of tools to test and 
build concurrent, parallel, and distributed code.

Enough talk about Go's marvelous concurrency features and tools, let's jump in.

The primary method of handling concurrency is through a goroutine. Admittedly, 
our first piece of concurrent code (mentioned in the preface) didn't do a whole 
lot, simply spitting out alternating "hello"s and "world"s until the entire task 
was complete.

Here is that code once again:

```go
package main

import (
  "fmt"
  "time"
)

type Job struct {
  i int
  max int
  text string
}

func outputText(j *Job) {
  for j.i < j.max {
    time.Sleep(1 * time.Millisecond)
    fmt.Println(j.text)
    j.i++
  }
}

func main() {
  hello := new(Job)
  world := new(Job)

  hello.text = "hello"
  hello.i = 0
  hello.max = 3
  
  world.text = "world"
  world.i = 0
  world.max = 5

  go outputText(hello)
  outputText(world)

}
```

### Tip

**Downloading the example code**

You can download the example code files for all Packt books you have purchased 
from your account at [http://www.  packtpub.com][1]. If you purchased this book 
elsewhere, you can visit <http://www.packtpub.com/support> and register to have 
the files e-mailed directly to you.

[1]: http://www.%20packtpub.com

But, if you think back to our real-world example of planning a surprise party 
for your grandmother, that's exactly how things often have to be managed with 
limited or finite resources. This asynchronous behavior is critical for some 
applications to run smoothly, although our example essentially ran in a vacuum.

You may have noticed one quirk in our early example: despite the fact that we 
called the `outputText()` function on the `hello` struct first, our output 
started with the `world` struct's text value. Why is that?

Being asynchronous, when a goroutine is invoked, it waits for the blocking code 
to complete before concurrency begins. You can test this by replacing the 
`outputText()` function call on the `world` struct with a goroutine, as shown in 
the following code:

```bash
  go outputText(hello)
  go outputText(world)
```

If you run this, you will get no output because the main function ends while the 
asynchronous goroutines are running. There are a couple of ways to stop this to 
see the output before the main function finishes execution and the program 
exits. The classic method simply asks for user input before execution, allowing 
you to directly control when the application finishes. You can also put an 
infinite loop at the end of your main function, as follows:

```go
for {}
```

Better yet, Go also has a built-in mechanism for this, which is the `WaitGroup` 
type in the `sync` package.

If you add a `WaitGroup` struct to your code, it can delay execution of the main 
function until after all goroutines are complete. In simple terms, it lets you 
set a number of required iterations to get a completed response from the 
goroutines before allowing the application to continue. Let's look at a minor 
modification to our "Hello World" application in the following section.

## A patient goroutine

From here, we'll implement a `WaitGroup` struct to ensure our goroutines run 
entirely before moving on with our application. In this case, when we say 
patient, it's in contrast to the way we've seen goroutines run outside of a 
parent method with our previous example. In the following code, we will 
implement our first `Waitgroup` struct:

```go
package main

import (
  "fmt"
  "sync"
  "time"
)

type Job struct {
  i int
  max int
  text string
}

func outputText(j *Job, goGroup *sync.WaitGroup) {
  for j.i < j.max {
    time.Sleep(1 * time.Millisecond)
    fmt.Println(j.text)
    j.i++
  }
  goGroup.Done()
}

func main() {

  goGroup := new(sync.WaitGroup)
  fmt.Println("Starting")

  hello := new(Job)
  hello.text = "hello"
  hello.i = 0
  hello.max = 2

  world := new(Job)
  world.text = "world"
  world.i = 0
  world.max = 2

  go outputText(hello, goGroup)
  go outputText(world, goGroup)

  goGroup.Add(2)
  goGroup.Wait()

}
```

Let's look at the changes in the following code:

```go
  goGroup := new(sync.WaitGroup)
```

Here, we declared a `WaitGroup` struct named `goGroup`. This variable will 
receive note that our goroutine function has completed *x* number of times 
before allowing the program to exit. Here's an example of sending such an 
expectation in `WaitGroup`:

```go
  goGroup.Add(2)
```

The `Add()` method specifies how many `Done` messages `goGroup` should receive 
before satisfying its wait. Here, we specified `2` because we have two functions 
running asynchronously. If you had three goroutine members and still called two, 
you may see the output of the third. If you added a value more than two to 
`goGroup`, for example, `goGroup.Add(3)`, then `WaitGroup` would wait forever 
and deadlock.

With that in mind, you shouldn't manually set the number of goroutines that need 
to wait; this is ideally handled computationally or explicitly in a range. This 
is how we tell `WaitGroup` to wait:

```go
  goGroup.Wait()
```

Now, we wait. This code will fail for the same reason `goGroup.Add(3)` failed; 
the `goGroup` struct never receives messages that our goroutines are done. So, 
let's do this as shown in the following code snippet:

```go
func outputText(j *Job, goGroup *sync.WaitGroup) {
  for j.i < j.max {
    time.Sleep(1 * time.Millisecond)
    fmt.Println(j.text)
    j.i++
  }
  goGroup.Done()
}
```

We've only made two changes to our `outputText()` function from the preface. 
First, we added a pointer to our `goGroup` as the second function argument. 
Then, when all our iterations were complete, we told `goGroup` that they are all 
done.


## Implementing the defer control mechanism

While we're here, we should take a moment and talk about defer. Go has an 
elegant implementation of the defer control mechanism. If you've used defer (or 
something functionally similar) in other languages, this will seem familiar—it's 
a useful way of delaying the execution of a statement until the rest of the 
function is complete.

For the most part, this is syntactical sugar that allows you to see related 
operations together, even though they won't execute together. If you've ever 
written something similar to the following pseudocode, you'll know what I mean:

```go
x = file.open('test.txt')
int longFunction() {
…
}
x.close();
```

You probably know the kind of pain that can come from large "distances" 
separating related bits of code. In Go, you can actually write the code similar 
to the following:

```go
package main

import(
"os"
)

func main() {
    file, _ := os.Create("/defer.txt")
    defer file.Close()
    for {
        break
    }
}
```

There isn't any actual functional advantage to this other than making clearer, 
more readable code, but that's a pretty big plus in itself. Deferred calls are 
executed reverse of the order in which they are defined, or last-in-first-out.  
You should also take note that any data passed by reference may be in an 
unexpected state.

For example, refer to the following code snippet:

```go
func main() {
  aValue := new(int)
  defer fmt.Println(*aValue)
  for i := 0; i < 100; i++ {
        *aValue++
  }
}
```

This will return `0`, and not `100`, as it is the default value for an integer.

### Note

*Defer* is not the same as *deferred* (or futures/promises) in other languages. 
We'll talk about Go's implementations and alternatives to futures and promises 
in [Chapter 2][1], *Understanding the Concurrency Model*.

[1]: ./ch02-understanding-the-concurrency-model.md


## Using Go's scheduler

With a lot of concurrent and parallel applications in other languages, the 
management of both soft and hard threads is handled at the operating system 
level. This is known to be inherently inefficient and expensive as the OS is 
responsible for context switching, among multiple processes. When an application 
or process can manage its own threads and scheduling, it results in faster 
runtime. The threads granted to our application and Go's scheduler have fewer OS 
attributes that need to be considered in context to switching, resulting in less 
overhead.

If you think about it, this is self-evident—the more you have to juggle, the 
slower it is to manage all of the balls. Go removes the natural inefficiency of 
this mechanism by using its own scheduler.

There's really only one quirk to this, one that you'll learn very early on: if 
you don't ever yield to the main thread, your goroutines will perform in 
unexpected ways (or won't perform at all).

Another way to look at this is to think that a goroutine must be blocked before 
concurrency is valid and can begin. Let's modify our example and include some 
file I/O to log to demonstrate this quirk, as shown in the following code:

```go
package main

import (
    "fmt"
    "time"
    "io/ioutil"
)

type Job struct {
    i int
    max int
    text string
}

func outputText(j *Job) {
    fileName := j.text + ".txt"
    fileContents := ""
    for j.i < j.max {
        time.Sleep(1 * time.Millisecond)
        fileContents += j.text
        fmt.Println(j.text)
        j.i++
    }
    err := ioutil.WriteFile(fileName, []byte(fileContents), 0644)
    if (err != nil) {
        panic("Something went awry")
    }
}

func main() {
    hello := new(Job)
    hello.text = "hello"
    hello.i = 0
    hello.max = 3

    world := new(Job)
    world.text = "world"
    world.i = 0
    world.max = 5


    go outputText(hello)
    go outputText(world)
}
```

In theory, all that has changed is that we're now using a file operation to log 
each operation to a distinct file (in this case, `hello.txt` and `world.txt`). 
However, if you run this, no files are created.

In our last example, we used a `sync.WaitSync` struct to force the main thread 
to delay execution until asynchronous tasks were complete. While this works (and 
elegantly), it doesn't really explain *why* our asynchronous tasks fail. As 
mentioned before, you can also utilize blocking code to prevent the main thread 
from completing before its asynchronous tasks.

Since the Go scheduler manages context switching, each goroutine must yield 
control back to the main thread to schedule all of these asynchronous tasks. 
There are two ways to do this manually. One method, and probably the ideal one, 
is the `WaitGroup` struct. Another is the `GoSched()` function in the runtime 
package.

The `GoSched()` function temporarily yields the processor and then returns to 
the current goroutine. Consider the following code as an example:

```go
package main

import(
    "runtime"
    "fmt"
)

func showNumber(num int) {
    fmt.Println(num)
}

func main() {
    iterations := 10
    
    for i := 0; i<=iterations; i++ {

      go showNumber(i)

    }
    // runtime.Gosched()
    fmt.Println("Goodbye!")

}
```

Run this with `runtime.Gosched()` commented out and the underscore before 
`"runtime"` removed, and you'll see only `Goodbye!`. This is because there's no 
guarantee as to how many goroutines, if any, will complete before the end of the 
`main()` function.

As we learned earlier, you can explicitly wait for a finite set number of 
goroutines before ending the execution of the application.  However, `Gosched()` 
allows (in most cases) for the same basic functionality. Remove the comment 
before `runtime.Gosched()`, and you should get 0 through 10 printed before 
`Goodbye!`.

Just for fun, try running this code on a multicore server and modify your max 
processors using `runtime.GOMAXPROCS()`, as follows:

```go
func main() {
    runtime.GOMAXPROCS(2)
```

Also, push your `runtime.Gosched()` to the absolute end so that all goroutines 
have a chance to run before `main` ends.

Got something unexpected? That's not unexpected! You may end up with a totally 
jostled execution of your goroutines, as shown in the following screenshot:

```
Goodbye!
2
1
0
5
4
3
7
9
8
6
10
```

Although it's not entirely necessary to demonstrate how juggling your goroutines 
with multiple cores can be vexing, this is one of the simplest ways to show 
exactly why it's important to have communication between them (and the Go 
scheduler).

You can debug the parallelism of this using `GOMAXPROCS > 1`, enveloping your 
goroutine call with a timestamp display, as follows:

```go
  tstamp := strconv.FormatInt(time.Now().UnixNano(), 10)
  fmt.Println(num, tstamp)
```

### Note

Remember to import the `time` and `strconv` parent packages here.

This will also be a good place to see concurrency and compare it to parallelism 
in action. First, add a one-second delay to the `showNumber()` function, as 
shown in the following code snippet:

```go
func showNumber(num int) {
    tstamp := strconv.FormatInt(time.Now().UnixNano(), 10)
    fmt.Println(num,tstamp)
    time.Sleep(time.Millisecond * 10)
}
```

Then, remove the goroutine call before the `showNumber()` function with 
`GOMAXPROCS(0)`, as shown in the following code snippet:

```go
    runtime.GOMAXPROCS(0)
    iterations := 10
    
    for i := 0; i<=iterations; i++ {
        showNumber(i)
    }
```

As expected, you get 0-10 with 10-millisecond delays between them followed by 
`Goodbye!` as an output. This is straight, serial computing.

Next, let's keep `GOMAXPROCS` at zero for a single thread, but restore the 
goroutine as follows:

```go
go showNumber(i)
```

This is the same process as before, except for the fact that everything will 
execute within the same general timeframe, demonstrating the concurrent nature 
of execution. Now, go ahead and change your `GOMAXPROCS` to two and run again. 
As mentioned earlier, there is only one (or possibly two) timestamp, but the 
order has changed because everything is running simultaneously.

Goroutines aren't (necessarily) thread-based, but they feel like they are. When 
Go code is compiled, the goroutines are multiplexed across available threads. 
It's this very reason why Go's scheduler needs to know what's running, what 
needs to finish before the application's life ends, and so on. If the code has 
two threads to work with, that's what it will use.


## Using system variables

So what if you want to know how many threads your code has made available to 
you?

Go has an environment variable returned from the runtime package function 
`GOMAXPROCS`. To find out what's available, you can write a quick application 
similar to the following code:

```go
package main

import (
    "fmt"
    "runtime"
)

func listThreads() int {
    threads := runtime.GOMAXPROCS(0)
    return threads
}

func main() {
    runtime.GOMAXPROCS(2)
    fmt.Printf("%d thread(s) available to Go.", listThreads())
}
```

A simple Go build on this will yield the following output:

```bash
2 thread(s) available to Go.
```

The `0` parameter (or no parameter) delivered to `GOMAXPROCS` means no change is 
made. You can put another number in there, but as you might imagine, it will 
only return what is actually available to Go. You cannot exceed the available 
cores, but you can limit your application to use less than what's available.

The `GOMAXPROCS()` call itself returns an integer that represents the *previous* 
number of processors available. In this case, we first set it to two and then 
set it to zero (no change), returning two.

It's also worth noting that increasing `GOMAXPROCS` can sometimes *decrease* the 
performance of your application.

As there are context-switching penalties in larger applications and operating 
systems, increasing the number of threads used means goroutines can be shared 
among more than one, and the lightweight advantage of goroutines might be 
sacrificed.

If you have a multicore system, you can test this pretty easily with Go's 
internal benchmarking functionality. We'll take a closer look at this 
functionality in [Chapter 5][2], *Locks, Blocks, and Better Channels,* and 
[Chapter 7][3], *Performance and Scalability*.

The runtime package has a few other very useful environment variable return 
functions, such as `NumCPU`, `NumGoroutine`, `CPUProfile`, and `BlockProfile`. 
These aren't just handy to debug, they're also good to know how to best utilize 
your resources. This package also plays well with the reflect package, which 
deals with metaprogramming and program self-analysis. We'll touch on that in 
more detail later in [Chapter 9][4], *Logging and Testing Concurrency in Go*, 
and [Chapter 10][5], *Advanced Concurrency and Best Practices*.

[2]: ./ch05-locks-blocks-and-better-channels.md
[3]: ./ch07-performace-and-scalability.md
[4]: ./ch09-logging-and-testing-concurrency-in-go.md
[5]: ./ch10-advanced-concurrency-and-best-pratices.md


## Understanding goroutines versus coroutines

At this point, you may be thinking, "Ah, goroutines, I know these as 
coroutines." Well, yes and no.

A coroutine is a cooperative task control mechanism, but in its most simplistic 
sense, a coroutine is not concurrent. While coroutines and goroutines are 
utilized in similar ways, Go's focus on concurrency provides a lot more than 
just state control and yields. In the examples we've seen so far, we have what 
we can call *dumb* goroutines. Although they operate in the same time and 
address space, there's no real communication between the two. If you look at 
coroutines in other languages, you may find that they are often not necessarily 
concurrent or asynchronous, but rather they are step-based. They yield to 
`main()` and to each other, but two coroutines might not necessarily communicate 
between each other, relying on a centralized, explicitly written data management 
system.

### Note

**The original coroutine**

Coroutines were first described for COBOL by Melvin Conway. In his paper, 
*Design of a Separable Transition-Diagram Compiler*, he suggested that the 
purpose of a coroutine was to take a program broken apart into subtasks and 
allow them to operate independently, sharing only small pieces of data.

Goroutines can sometimes violate the basic tenets of Conway's coroutines. For 
example, Conway suggested that there should be only a unidirectional path of 
execution; in other words, A followed by B, then C, and then D, and so on, where 
each represents an application chunk in a coroutine. We know that goroutines can 
be run in parallel and can execute in a seemingly arbitrary order (at least 
without direction). To this point, our goroutines have not shared any 
information either; they've simply executed in a shared pattern.

## Implementing channels


So far, we've dabbled in concurrent processes that are capable of doing a lot 
but not effectively communicating with each other. In other words, if you have 
two processes occupying the same processing time and sharing the same memory and 
data, you must have a way of knowing which process is in which place as part of 
a larger task.

Take, for example, an application that must loop through one paragraph of Lorem 
Ipsum and capitalize each letter, then write the result to a file. Of course, we 
will not really need a concurrent application to do this (and in fact, it's an 
endemic function of almost any language that handles strings), but it's a quick 
way to demonstrate the potential limitations of isolated goroutines. Shortly, 
we'll turn this primitive example into something more practical, but for now, 
here's the beginning of our capitalization example:

```go
package main

import (
  "fmt"
  "runtime"
  "strings"
)
var loremIpsum string
var finalIpsum string
var letterSentChan chan string

func deliverToFinal(letter string, finalIpsum *string) {
    *finalIpsum += letter
}

func capitalize(current *int, length int, letters []byte, 
    finalIpsum *string) {
    for *current < length {
        thisLetter := strings.ToUpper(string(letters[*current]))

        deliverToFinal(thisLetter, finalIpsum)
        *current++
    }
}

func main() {
    runtime.GOMAXPROCS(2)

    index := new(int)
    *index = 0
    loremIpsum = "Lorem ipsum dolor sit amet, consectetur adipiscing 
    elit. Vestibulum venenatis magna eget libero tincidunt, ac 
    condimentum enim auctor. Integer mauris arcu, dignissim sit amet 
    convallis vitae, ornare vel odio. Phasellus in lectus risus. Ut 
    sodales vehicula ligula eu ultricies. Fusce vulputate fringilla 
    eros at congue. Nulla tempor neque enim, non malesuada arcu 
    laoreet quis. Aliquam eget magna metus. Vivamus lacinia 
    venenatis dolor, blandit faucibus mi iaculis quis. Vestibulum 
    sit amet feugiat ante, eu porta justo."

    letters := []byte(loremIpsum)
    length := len(letters)

    go capitalize(index, length, letters, &finalIpsum)
    go func() {
        go capitalize(index, length, letters, &finalIpsum)
    }()

    fmt.Println(length, " characters.")
    fmt.Println(loremIpsum)
    fmt.Println(*index)
    fmt.Println(finalIpsum)
```

If we run this with some degree of parallelism here but no communication between 
our goroutines, we'll end up with a jumbled mess of text, as shown in the 
following screenshot:

![Implementing
channels](/library/view/mastering-concurrency-in/9781783983483/graphics/3483OS_01_02.jpg)

Due to the demonstrated unpredictability of concurrent scheduling in Go, it may 
take many iterations to get this exact output. In fact, you may never get the 
exact output.

This won't do, obviously. So how do we best structure this application?  The 
missing piece here is synchronization, but we could also do with a better design 
pattern.

Here's another way to break this problem down into pieces. Instead of having two 
processes handling the same thing in parallel, which is rife with risk, let's 
have one process that takes a letter from the `loremIpsum` string and 
capitalizes it, and then pass it onto another process to add it to our 
`finalIpsum` string.

You can envision this as two people sitting at two desks, each with a stack of 
letters. Person A is responsible to take a letter and capitalize it. He then 
passes the letter onto person B, who then adds it to the `finalIpsum` stack. To 
do this, we'll implement a channel in our code in an application tasked with 
taking text (in this case, the first line of Abraham Lincoln's Gettysburg 
address) and capitalizing each letter.

## Channel-based sorting at the letter capitalization factory

Let's take the last example and do something (slightly) more purposeful by 
attempting to capitalize the preamble of Abraham Lincoln's Gettysburg address 
while mitigating the sometimes unpredictable effect of concurrency in Go, as 
shown in the following code:

```go
package main

import(
    "fmt"
    "sync"
    "runtime"
    "strings"
)

var initialString string
var finalString string

var stringLength int

func addToFinalStack(letterChannel chan string, wg 
    *sync.WaitGroup) {
    letter := <-letterChannel
    finalString += letter
    wg.Done()
}


func capitalize(letterChannel chan string, currentLetter string, wg *sync.WaitGroup) {
    thisLetter := strings.ToUpper(currentLetter)
    wg.Done()
    letterChannel <- thisLetter  
}

func main() {
    runtime.GOMAXPROCS(2)
    var wg sync.WaitGroup

    initialString = "Four score and seven years ago our fathers 
    brought forth on this continent, a new nation, conceived in 
    Liberty, and dedicated to the proposition that all men are 
    created equal."
    initialBytes := []byte(initialString)

    var letterChannel chan string = make(chan string)

    stringLength = len(initialBytes)

    for i := 0; i < stringLength; i++ {
        wg.Add(2)

        go capitalize(letterChannel, string(initialBytes[i]), &wg)
        go addToFinalStack(letterChannel, &wg)

        wg.Wait()
    }

    fmt.Println(finalString)
}
```

You'll note that we even bumped this up to a duo-core process and ended up with 
the following output:

```bash
go run alpha-channel.go
FOUR SCORE AND SEVEN YEARS AGO OUR FATHERS BROUGHT FORTH ON THIS 
  CONTINENT, A NEW NATION, CONCEIVED IN LIBERTY, AND DEDICATED TO THE 
  PROPOSITION THAT ALL MEN ARE CREATED EQUAL.
```

The output is just as we expected. It's worth reiterating that this example is 
overkill of the most extreme kind, but we'll parlay this functionality into a 
usable practical application shortly.

So what's happening here? First, we reimplemented the `sync.WaitGroup` struct to 
allow all of our concurrent code to execute while keeping the main thread alive, 
as shown in the following code snippet:

```go
var wg sync.WaitGroup
...
for i := 0; i < stringLength; i++ {
    wg.Add(2)

    go capitalize(letterChannel, string(initialBytes[i]), &wg)
    go addToFinalStack(letterChannel, &wg)

    wg.Wait()
}
```

We allow each goroutine to tell the `WaitGroup` struct that we're done with the 
step. As we have two goroutines, we queue two `Add()` methods to the `WaitGroup` 
struct. Each goroutine is responsible to announce that it's done.

Next, we created our first channel. We instantiate a channel with the following 
line of code:

```go
  var letterChannel chan string = make(chan string)
```

This tells Go that we have a channel that will send and receive a string to 
various procedures/goroutines. This is essentially the manager of all of the 
goroutines. It is also responsible to send and receive data to goroutines and 
manage the order of execution. As we mentioned earlier, the ability of channels 
to operate with internal context switching and without reliance on 
multithreading permits them to operate very quickly.

There is a built-in limit to this functionality. If you design non-concurrent or 
blocking code, you will effectively remove concurrency from goroutines. We will 
talk more about this shortly.

We run two separate goroutines through `letterChannel`: `capitalize()` and 
`addToFinalStack()`. The first one simply takes a single byte from a byte array 
constructed from our string and capitalizes it. It then returns the byte to the 
channel as shown in the following line of code:

```go
letterChannel <- thisLetter
```

All communication across a channel happens in this fashion. The `<-` symbol 
syntactically tells us that data will be sent back to (or back through) a 
channel. It's never necessary to do anything with this data, but the most 
important thing to know is that a channel can be blocking, at least per thread, 
until it receives data back. You can test this by creating a channel and then 
doing absolutely nothing of value with it, as shown in the following code 
snippet:

```go
package main

func doNothing()(string) {
    return "nothing"
}

func main() {
    var channel chan string = make(chan string)
    channel <- doNothing()
}
```

As nothing is sent along the channel and no goroutine is instantiated, this 
results in a deadlock. You can fix this easily by creating a goroutine and by 
bringing the channel into the global space by creating it outside of `main()`.

### Note

For the sake of clarity, our example here uses a local scope channel.  Keeping 
these global whenever possible removes a lot of cruft, particularly if you have 
a lot of goroutines, as references to the channel can clutter up your code in a 
hurry.

For our example as a whole, you can look at it as is shown in the following 
figure:

![Channel-based sorting at the letter capitalization
factory](/library/view/mastering-concurrency-in/9781783983483/graphics/3483OS_01_03.jpg)

## Cleaning up our goroutines

You may be wondering why we need a `WaitGroup` struct when using channels. After 
all, didn't we say that a channel gets blocked until it receives data?  This is 
true, but it requires one other piece of syntax.

A nil or uninitialized channel will always get blocked. We will discuss the 
potential uses and pitfalls of this in [Chapter 7][6], *Performance and 
Scalability*, and [Chapter 10][5], *Advanced Concurrency and Best Practices*.

[6]: ./ch07-performace-and-scalability.md

You have the ability to dictate how a channel blocks the application based on a 
second option to the `make` command by dictating the channel buffer.

### Buffered or unbuffered channels

By default, channels are unbuffered, which means they will accept anything sent 
on them if there is a channel ready to receive. It also means that every channel 
call will block the execution of the application. By providing a buffer, the 
channel will only block the application when many returns have been sent.

A buffered channel is synchronous. To guarantee asynchronous performance, you'll 
want to experiment by providing a buffer length. We'll look at ways to ensure 
our execution falls as we expect in the next chapter.

### Note

Go's channel system is based on **Communicating Sequential Processes** 
(**CSP**), a formal language to design concurrent patterns and multiprocessing. 
You will likely encounter CSP on its own when people describe goroutines and 
channels.

## Using the select statement

One of the issues with first implementing channels is that whereas goroutines 
were formerly the method of simplistic and concurrent execution of code, we now 
have a single-purpose channel that dictates application logic across the 
goroutines. Sure, the channel is the traffic manager, but it never knows when 
traffic is coming, when it's no longer coming, and when to go home, unless being 
explicitly told. It waits passively for communication and can cause problems if 
it never receives any.

Go has a select control mechanism, which works just as effectively as a `switch` 
statement does, but on channel communication instead of variable values. A 
`switch` statement modifies execution based on the value of a variable, and 
`select` reacts to actions and communication across a channel. You can use this 
to orchestrate and arrange the control flow of your application. The following 
code snippet is our traditional `switch`, familiar to Go users and common among 
other languages:

```go
switch {
  case 'x':
  case 'y':
}
```

The following code snippet represents the `select` statement:

```go
select {
  case <- channelA:
  case <- channelB:
}
```

In a `switch` statement, the right-hand expression represents a value; in 
`select`, it represents a receive operation on a channel. A `select` statement 
will block the application until some information is sent along the channel. If 
nothing is sent ever, the application deadlocks and you'll get an error to that 
effect.

If two receive operations are sent at the same time (or if two cases are 
otherwise met), Go will evaluate them in an unpredictable fashion.

So, how might this be useful? Let's look at a modified version of the letter 
capitalization application's main function:

```go
package main

import(
    "fmt"  
    "strings"
)

var initialString string
var initialBytes []byte
var stringLength int
var finalString string
var lettersProcessed int
var applicationStatus bool
var wg sync.WaitGroup

func getLetters(gQ chan string) {
    for i := range initialBytes {
        gQ <- string(initialBytes[i])  
    }
}

func capitalizeLetters(gQ chan string, sQ chan string) {
    for {
        if lettersProcessed >= stringLength {
            applicationStatus = false
            break
        }
        select {
        case letter := <- gQ:
            capitalLetter := strings.ToUpper(letter)
            finalString += capitalLetter
            lettersProcessed++
        }
    }
}

func main() {
    applicationStatus = true;

    getQueue := make(chan string)
    stackQueue := make(chan string)

    initialString = "Four score and seven years ago our fathers brought forth on this continent, a new nation, conceived in Liberty, and dedicated to the proposition that all men are created equal."
    initialBytes = []byte(initialString)
    stringLength = len(initialString)
    lettersProcessed = 0

    fmt.Println("Let's start capitalizing")

    go getLetters(getQueue)
    capitalizeLetters(getQueue,stackQueue)

    close(getQueue)
    close(stackQueue)

    for {
        if applicationStatus == false {
            fmt.Println("Done")
            fmt.Println(finalString)
            break
        }
    }
}
```

The primary difference here is we now have a channel that listens for data 
across two functions running concurrently, `getLetters` and `capitalizeLetters`. 
At the bottom, you'll see a `for{}` loop that keeps the main active until the 
`applicationStatus` variable is set to `false`. In the following code, we pass 
each of these bytes as a string through the Go channel:

```go
func getLetters(gQ chan string) {
    for i := range initialBytes {
        gQ <- string(initialBytes[i])  
    }
}
```

The `getLetters` function is our primary goroutine that fetches individual 
letters from the byte array constructed from Lincoln's line. As the function 
iterates through each byte, it sends the letter through the `getQueue` channel.

On the receiving end, we have `capitalizeLetters` that takes each letter as it's 
sent across the channel, capitalizes it, and appends to our `finalString` 
variable. Let's take a look at this:

```go
func capitalizeLetters(gQ chan string, sQ chan string) {
    for {
        if lettersProcessed >= stringLength {
            applicationStatus = false
            break
        }
        select {
        case letter := <- gQ:
            capitalLetter := strings.ToUpper(letter)
            finalString += capitalLetter
            lettersProcessed++
        }
    }
}
```

It's critical that all channels are closed at some point or our application will 
hit a deadlock. If we never break the `for` loop here, our channel will be left 
waiting to receive from a concurrent process, and the program will deadlock. We 
manually check to see that we've capitalized all letters and only then break the 
loop.


## Closures and goroutines

You may have noticed the anonymous goroutine in Lorem Ipsum:

```go
    go func() {
        go capitalize(index, length, letters, &finalIpsum)
    }()
```

While it isn't always ideal, there are plenty of places where inline functions 
work best in creating a goroutine.

The easiest way to describe this is to say that a function isn't big or 
important enough to deserve a named function, but the truth is, it's more about 
readability. If you have dealt with lambdas in other languages, this probably 
doesn't need much explanation, but try to reserve these for quick inline 
functions.

In the earlier examples, the closure works largely as a wrapper to invoke a 
`select` statement or to create anonymous goroutines that will feed the `select` 
statement.

Since functions are first-class citizens in Go, not only can you utilize inline 
or anonymous functions directly in your code, but you can also pass them to and 
from other functions.

Here's an example that passes a function's result as a return value, keeping the 
state resolute outside of that returned function. In this, we'll return a 
function as a variable and iterate initial values on the returned function. The 
initial argument will accept a string that will be trimmed by word length with 
each successive call of the returned function.

```go
import(
    "fmt"
    "strings"
)

func shortenString(message string) func() string {
    return func() string {
        messageSlice := strings.Split(message," ")
        wordLength := len(messageSlice)
        if wordLength < 1 {
            return "Nothingn Left!"
        }else {
            messageSlice = messageSlice[:(wordLength-1)]
            message = strings.Join(messageSlice, " ")
            return message
        }
    }
}

func main() {
    myString := shortenString("Welcome to concurrency in Go! ...")

    fmt.Println(myString())
    fmt.Println(myString())  
    fmt.Println(myString())  
    fmt.Println(myString())  
    fmt.Println(myString())  
    fmt.Println(myString())
}
```

Once initialized and returned, we set the message variable, and each successive 
run of the returned method iterates on that value. This functionality allows us 
to eschew running a function multiple times on returned values or loop 
unnecessarily when we can very cleanly handle this with a closure as shown.


## Building a web spider using goroutines and channels

Let's take the largely useless capitalization application and do something 
practical with it.  Here, our goal is to build a rudimentary spider. In doing 
so, we'll accomplish the following tasks:

  - Read five URLs
  - Read those URLs and save the contents to a string
  - Write that string to a file when all URLs have been scanned and read

These kinds of applications are written every day, and they're the ones that 
benefit the most from concurrency and non-blocking code.

It probably goes without saying, but this is not a particularly elegant web 
scraper. For starters, it only knows a few start points—the five URLs that we 
supply it. Also, it's neither recursive nor is it thread-safe in terms of data 
integrity.

That said, the following code works and demonstrates how we can use channels and 
the `select` statements:

```go
package main

import(
    "fmt"
    "io/ioutil"
    "net/http"
    "time"
)

var applicationStatus bool
var urls []string
var urlsProcessed int
var foundUrls []string
var fullText string
var totalURLCount int
var wg sync.WaitGroup

var v1 int
```

First, we have our most basic global variables that we'll use for the 
application state. The `applicationStatus` variable tells us that our spider 
process has begun and `urls` is our slice of simple string URLs.  The rest are 
idiomatic data storage variables and/or application flow mechanisms. The 
following code snippet is our function to read the URLs and pass them across the 
channel:

```go
func readURLs(statusChannel chan int, textChannel chan string) {
    time.Sleep(time.Millisecond * 1)
    fmt.Println("Grabbing", len(urls), "urls")
    for i := 0; i < totalURLCount; i++ {
        fmt.Println("Url", i, urls[i])
        resp, _ := http.Get(urls[i])
        text, err := ioutil.ReadAll(resp.Body)

        textChannel <- string(text)

        if err != nil {
          fmt.Println("No HTML body")
        }

        statusChannel <- 0
    }
}
```

The `readURLs` function assumes `statusChannel` and `textChannel` for 
communication and loops through the `urls` variable slice, returning the text on 
`textChannel` and a simple ping on `statusChannel`. Next, let's look at the 
function that will append scraped text to the full text:

```go
func addToScrapedText(textChannel chan string, processChannel chan bool) {
    for {
        select {
        case pC := <-processChannel:
            if pC == true {
                // hang on
            }
            if pC == false {

                close(textChannel)
                close(processChannel)
            }
        case tC := <-textChannel:
            fullText += tC
        }
    }
}
```

We use the `addToScrapedText` function to accumulate processed text and add it 
to a master text string. We also close our two primary channels when we get a 
kill signal on our `processChannel`. Let's take a look at the `evaluateStatus()` 
function:

```go
func evaluateStatus(statusChannel chan int, textChannel chan string, processChannel chan bool) {
    for {
        select {
        case status := <-statusChannel:
            fmt.Print(urlsProcessed, totalURLCount)
            urlsProcessed++
            if status == 0 {
                fmt.Println("Got url")
            }
            if status == 1 {
                close(statusChannel)
            }
            if urlsProcessed == totalURLCount {
                fmt.Println("Read all top-level URLs")
                processChannel <- false
                applicationStatus = false
            }
        }
    }
}
```

At this juncture, all that the `evaluateStatus` function does is determine 
what's happening in the overall scope of the application. When we send a `0` 
(our aforementioned ping) through this channel, we increment our `urlsProcessed` 
variable. When we send a `1`, it's a message that we can close the channel. 
Finally, let's look at the `main` function:

```go
func main() {
    applicationStatus = true
    statusChannel := make(chan int)
    textChannel := make(chan string)
    processChannel := make(chan bool)
    totalURLCount = 0

    urls = append(urls, "http://www.mastergoco.com/index1.html")
    urls = append(urls, "http://www.mastergoco.com/index2.html")
    urls = append(urls, "http://www.mastergoco.com/index3.html")
    urls = append(urls, "http://www.mastergoco.com/index4.html")
    urls = append(urls, "http://www.mastergoco.com/index5.html")

    fmt.Println("Starting spider")

    urlsProcessed = 0
    totalURLCount = len(urls)

    go evaluateStatus(statusChannel, textChannel, processChannel)
    go readURLs(statusChannel, textChannel)
    go addToScrapedText(textChannel, processChannel)

    for {
        if applicationStatus == false {
            fmt.Println(fullText)
            fmt.Println("Done!")
            break
        }
        select {
        case sC := <-statusChannel:
            fmt.Println("Message on StatusChannel", sC)
        }
    }
}
```

This is a basic extrapolation of our last function, the capitalization function. 
However, each piece here is responsible for some aspect of reading URLs or 
appending its respective content to a larger variable.

In the following code, we created a sort of master loop that lets you know when 
a URL has been grabbed on `statusChannel`:

```go
    for {
        if applicationStatus == false {
            fmt.Println(fullText)
            fmt.Println("Done!")
            break
        }
        select {
        case sC := <- statusChannel:
            fmt.Println("Message on StatusChannel",sC)
        }
    }
```

Often, you'll see this wrapped in `go func()` as part of a `WaitGroup` struct, 
or not wrapped at all (depending on the type of feedback you require).

The control flow, in this case, is `evaluateStatus`, which works as a channel 
monitor that lets us know when data crosses each channel and ends execution when 
it's complete. The `readURLs` function immediately begins reading our URLs, 
extracting the underlying data and passing it on to `textChannel`.  At this 
point, our `addToScrapedText` function takes each sent HTML file and appends it 
to the `fullText` variable.  When `evaluateStatus` determines that all URLs have 
been read, it sets `applicationStatus` to `false`. At this point, the infinite 
loop at the bottom of `main()` quits.

As mentioned, a crawler cannot come more rudimentary than this, but seeing a 
real-world example of how goroutines can work in congress will set us up for 
safer and more complex examples in the coming chapters.

## Summary

In this chapter, we learned how to go from simple goroutines and instantiating 
channels to extending the basic functionality of goroutines and allowing cross-
channel, bidirectional communication within concurrent processes. We looked at 
new ways to create blocking code to prevent our main process from ending before 
our goroutines. Finally, we learned about using select statements to develop 
reactive channels that are silent unless data is sent along a channel.

In our rudimentary web spider example, we employed these concepts together to 
create a safe, lightweight process that could extract all links from an array of 
URLs, grab the content via HTTP, and store the resulting response.

In the next chapter, we'll go beneath the surface to see how Go's internal 
scheduling manages concurrency and start using channels to really utilize the 
power, thrift, and speed of concurrency in Go.

