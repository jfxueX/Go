[Concrrency in Go: goroutines (part 1)][1]

[1]: https://medium.com/fstack/concrrency-in-go-goroutines-part-1

Go supports traditional concurrent programming approach — memory access 
synchronization. However it heavily designed around CSP- *Communicating 
Sequential Processes.* This is actually great! Being able to choose between CSP 
primitives and memory access synchronizations is great for you since it gives 
you more control over what style of concurrent code you choose to write to solve 
problems. At the same time it is a little confusing.

#### Why build concurrency on the ideas of CSP?

Go team answer to the question as below:

> Concurrency and multi-threaded programming have a reputation for difficulty. We 
> believe this is due partly to complex designs such as pthreads and partly to 
> overemphasis on low-level details such as mutexes, condition variables, and 
> memory barriers. Higher-level interfaces enable much simpler code, even if there 
> are still mutexes and such under the covers.
 
> One of the most successful models for providing high-level linguistic support 
> for concurrency comes from Hoare’s Communicating Sequential Processes, or CSP. 
> Occam and Erlang are two well known languages that stem from CSP. Go’s 
> concurrency primitives derive from a different part of the family tree whose 
> main contribution is the powerful notion of channels as first class objects. 
> Experience with several earlier languages has shown that the CSP model fits well 
> into a procedural language framework.

Go has no notion of threads. Instead it introduces a term *goroutine*.  Put very 
simply, a *goroutine* is a function that is running concurrently alongside other 
parts of the source code. Every Go program has at least one (the main) 
*goroutine*. Starting a *goroutine* is very simple. All you need are placing 
*go* keyword before a function as shown example(*code 1*) below:

```go
import "fmt"

func main() {
    //.....
    go printHelloWorld()
    //...
}

func printHelloWorld() {
    fmt.Println("Hello World")
}
```

Or you can start anonymous function in another *goroutine* directly (*code 2*):

```go
import "fmt"

func main() {
    go func() {
       fmt.Println("Hello World")
    }()
}
```

So what is actually happening behind the scenes? What is a goroutine if it is 
not a thread? Why Go introduced them? Let’s see another note from official FAQs 
below:

> **Why goroutines instead of threads?**
> 
> Goroutines are part of making concurrency easy to use. The idea, which has been 
> around for a while, is to multiplex independently executing functions — 
> coroutines — onto a set of threads. When a coroutine blocks, such as by calling 
> a blocking system call, the run-time automatically moves other coroutines on the 
> same operating system thread to a different, runnable thread so they won’t be 
> blocked. The programmer sees none of this, which is the point. The result, which 
> we call goroutines, can be very cheap: they have little overhead beyond the 
> memory for the stack, which is just a few kilobytes.
> 
> To make the stacks small, Go’s run-time uses resizable, bounded stacks. A newly 
> minted goroutine is given a few kilobytes, which is almost always enough. When 
> it isn’t, the run-time grows (and shrinks) the memory for storing the stack 
> automatically, allowing many goroutines to live in a modest amount of memory. 
> The CPU overhead averages about three cheap instructions per function call. It 
> is practical to create hundreds of thousands of goroutines in the same address 
> space. If goroutines were just threads, system resources would run out at a much 
> smaller number.

Goroutines are not operation system’s threads, and they’re not also threads that 
are managed by a language’s runtime (aka green threads).  They’re a higher level 
of abstraction known of [*coroutines*][2] *and* can be considered a special 
class of coroutine. They deeply integrated to GO’s runtime, therefore suspension 
of a goroutine when gets blocked and resuming execution when unblocked totally 
observed and managed by the runtime. Go’s scheduler uses M:N principle for 
hosting goroutines. It maps M green threads to N OS threads and schedules 
goroutines onto the green threads. It distributes goroutines across the 
available threads.  If a goroutine blocks a green thread, in order to prevent 
blocking other goroutines the scheduler moves them to available green threads. 
Main problem with OS threads is that *context switching* is expensive process. 
Each thread has information about it’s current state. Every time saving this 
data to switch to another thread and restoring it when it’s time to run for the 
thread can be quite costly. What if we have too many concurrent process? Context 
switching will be main job of CPU! Go manages context switching managed on 
software side (Go’s scheduler) by using goroutines. This is much cheaper than 
doing it on OS side

[2]: https://en.wikipedia.org/wiki/Coroutine

Go follows a model of concurrency called the *fork-join* model. The word *fork* 
refers to the fact that at any point in the program, it can split off a *child* 
branch of execution to be run concurrently with its *parent*. The word *join* 
refers to the fact that at some point in the future, these concurrent branches 
of execution will join back together.  Where the child rejoins the parent is 
called a *join point*. Let’s look at example below (code 3):

```go
import (
    "fmt"
)

func main() {
    go printTime()                 // fork point
    fmt.Println("main goroutine")  
}

func printTime() {
    fmt.Println("second goroutine")
}
```

```
______________________________________
: main goroutine
```

Here, the *printTime* function will be run on its own goroutine and the main 
goroutine continues executing. In this example, there is no join point. It’s 
undetermined whether the *printTime* function will ever run or not. New 
goroutine will be created ** and scheduled to host *printTime* function with 
Go’s runtime to execute, but it may not actually get a chance to run before the 
main goroutine exits. In order to a create a join point, you have to synchronize 
goroutines. To do this we will use *WaitGroup* from the [sync][3] package(we 
will implement by using chanels in the next part. Loog at example below (*code 
4*):

[3]: https://golang.org/pkg/sync/

```go
import (
 "sync"
 "fmt"
)

func main() {
   wg:= sync.WaitGroup{}
   wg.Add(1)

   // fork point
   go func() {
       defer wg.Done()
       fmt.Println("second goroutine")
   }()
 
   // join point
   wg.Wait()
   fmt.Println("main goroutine")
}
```

```
_______________________________________
: second goroutine
: main goroutine
```

Here we deterministically block the main goroutine until the goroutine hosting 
the anonymous function terminates, prints “second goroutine” and then main 
gorouitine continues executing. Now we are able to print both texts.

### Conguratulation!!!

Now you know what is a goroutine and how to use it.
