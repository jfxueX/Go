[Part 23: Buffered Channels and Worker Pools][1]
================================================

02 August 2017

[Golang tutorial series][2]

[1]: https://golangbot.com/buffered-channels-worker-pools/
[2]: https://golangbot.com/learn-golang-series/
[3]: gb-channels.md


* [What are buffered channels?](#what-are-buffered-channels)
    * [Example](#example)
    * [Another Example](#another-example)
    * [Deadlock](#deadlock)
    * [Length vs Capacity](#length-vs-capacity)
    * [WaitGroup](#waitgroup)
    * [Worker Pool Implementation](#worker-pool-implementation)


## What are buffered channels?

All the channels we discussed in the [previous tutorial][3] were basically 
unbuffered. As we discussed in the [channels][3] tutorial in detail, sends and 
receives to an unbuffered channel are blocking.

It is possible to create a channel with a buffer. *Sends to a buffered channel 
are blocked only when the buffer is full. Similarly receives from a buffered 
channel are blocked only when the buffer is empty.*

Buffered channels can be created by passing an additional capacity parameter to 
the `make` function which specifies the size of the buffer.

```go
ch := make(chan type, capacity)  
```

*capacity* in the above syntax should be greater than 0 for a channel to have a 
buffer. The capacity for an unbuffered channel is 0 by default and hence we 
omitted the capacity parameter while creating channels in the [previous 
tutorial][3].

Lets write some code and create a buffered channel.

### Example

```go
package main

import (  
    "fmt"
)


func main() {  
    ch := make(chan string, 2)
    ch <- "naveen"
    ch <- "paul"
    fmt.Println(<- ch)
    fmt.Println(<- ch)
}
```

[Run program in playground](https://play.golang.org/p/It-em11etK)

In the program above, in line no. 9 we create a buffered channel with a capacity 
of 2. Since the channel has a capacity of 2, it is possible to write 2 strings 
into the channel without being blocked. We write 2 strings to the channel in 
line no. 10 and 11 and the channel does not block. We read the 2 strings written 
in line nos. 12 and 13 respectively. This program prints,

``` 
naveen  
paul  
```

### Another Example

Lets look at one more example of buffered channel in which the values to the 
channel are written in a concurrent Goroutine and read from the main Goroutine. 
This example will help us better understand when writes to a buffered channel 
block.

```go
package main

import (  
    "fmt"
    "time"
)

func write(ch chan int) {  
    for i := 0; i < 5; i++ {
        ch <- i
        fmt.Println("successfully wrote", i, "to ch")
    }
    close(ch)
}

func main() {  
    ch := make(chan int, 2)
    go write(ch)
    time.Sleep(2 * time.Second)
    for v := range ch {
        fmt.Println("read value", v,"from ch")
        time.Sleep(2 * time.Second)
    }
}
```

[Run program in playground](https://play.golang.org/p/bKe5GdgMK9)


In the program above, a buffered channel `ch` of capacity `2` is created in line 
no. 16 of the `main` Goroutine and passed to the `write` Goroutine in line no. 
17. Then the main Goroutine sleeps for 2 seconds.  During this time, the `write` 
Goroutine is running concurrently. The `write` Goroutine has a `for` loop which 
writes numbers from 0 to 4 to the `ch` channel. The capacity of this buffered 
channel is `2` and hence the write `Goroutine` will be able to write values `0` 
and `1` to the `ch` channel immediately and then it blocks until at least one 
value is read from `ch` channel. So this program will print the following 2 
lines immediately.

``` 
successfully wrote 0 to ch  
successfully wrote 1 to ch  
```

After printing the above two lines, the writes to the `ch` channel in the 
`write` Goroutine are blocked until someone reads from the `ch` channel. Since 
the main Goroutine sleeps for 2 seconds before starting to read from the 
channel, the program will not print anything for the next 2 seconds. The `main` 
Goroutine wakes up after 2 seconds and starts reading from the `ch` channel 
using a `for range` loop in line no. 19, prints the read value and then sleeps 
for 2 seconds again and this cycle continues until the `ch` is closed. So the 
program will print the following lines after 2 seconds,

``` 
read value 0 from ch  
successfully wrote 2 to ch  
```

This will continue until all values are written to the channel and it is closed 
in the `write` Goroutine. The final output would be,

``` 
successfully wrote 0 to ch  
successfully wrote 1 to ch  
read value 0 from ch  
successfully wrote 2 to ch  
read value 1 from ch  
successfully wrote 3 to ch  
read value 2 from ch  
successfully wrote 4 to ch  
read value 3 from ch  
read value 4 from ch  
```

### Deadlock

```go
package main

import (  
    "fmt"
)

func main() {  
    ch := make(chan string, 2)
    ch <- "naveen"
    ch <- "paul"
    ch <- "steve"
    fmt.Println(<-ch)
    fmt.Println(<-ch)
}
```

[Run program in playground](https://play.golang.org/p/FW-LHeH7oD)

In the program above, we write 3 strings to a buffered channel of capacity 2. 
When the control reaches the third write in line no. 11, the write is blocked 
since the channel has exceeded its capacity. Now some Goroutine must read from 
the channel in order for the write to proceed, but in this case there is no 
concurrent routine reading from this channel. Hence there will be a **deadlock** 
and the program will panic at run time with the following message,

```
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:  
main.main()  
    /tmp/sandbox274756028/main.go:11 +0x100
```

### Length vs Capacity

The capacity of a buffered channel is the number of values that the channel can 
hold. This is the value we specify when creating the buffered channel using the 
`make` function.

The length of the buffered channel is the number of elements currently queued in 
it.

A program will make things clear 😀

```go
package main

import (  
    "fmt"
)

func main() {  
    ch := make(chan string, 3)
    ch <- "naveen"
    ch <- "paul"
    fmt.Println("capacity is", cap(ch))
    fmt.Println("length is", len(ch))
    fmt.Println("read value", <-ch)
    fmt.Println("new length is", len(ch))
}
```

[Run program in playground](https://play.golang.org/p/2ggC64yyvr)

In the program above, the channel is created with a capacity of `3`, that is, it 
can hold 3 strings. We then write 2 strings to the channel in line nos. 9 and 10 
respectively. Now the channel has 2 strings queued in it and hence its length is 
`2`. In line no. 13, we read a string from the channel. Now the channel has only 
one string queued in it and hence its length becomes `1`. This program will 
print,

```
capacity is 3  
length is 2  
read value naveen  
new length is 1
```

### WaitGroup

The next section in this tutorial is about *Worker Pools*. To understand worker 
pools, we need to first know about `WaitGroup` as it will be used in the 
implementation of Worker pool.

A WaitGroup is used to wait for a collection of Goroutines to finish executing. 
The control is blocked until all Goroutines finish executing.  Lets say we have 
3 concurrently executing Goroutines spawned from the `main` Goroutine. The 
`main` Goroutines needs to wait for the 3 other Goroutines to finish before 
terminating. This can be accomplished using WaitGroup.


Lets stop the theory and write some code right away 😀

```go
package main

import (  
    "fmt"
    "sync"
    "time"
)

func process(i int, wg *sync.WaitGroup) {  
    fmt.Println("started Goroutine ", i)
    time.Sleep(2 * time.Second)
    fmt.Printf("Goroutine %d ended\n", i)
    wg.Done()
}

func main() {  
    no := 3
    var wg sync.WaitGroup
    for i := 0; i < no; i++ {
        wg.Add(1)
        go process(i, &wg)
    }
    wg.Wait()
    fmt.Println("All go routines finished executing")
}
```

[Run in playground](https://play.golang.org/p/CZNtu8ktQh)

[WaitGroup][4] is a struct type and we are creating a zero value variable of 
type `WaitGroup` in line no.18.  The way `WaitGroup` works is by using a 
counter. When we call `Add` on the `WaitGroup` and pass it an `int`, the 
`WaitGroup`'s counter is incremented by the value passed to `Add`. The way to 
decrement the counter is by calling `Done()` method on the WaitGroup. The 
`Wait()` method blocks the `Goroutine` in which it's called until the counter 
becomes zero.

[4]: https://golang.org/pkg/sync/#WaitGroup

In the above program, we call `wg.Add(1)` in line no. 20 inside the `for` loop 
which iterates 3 times. So the counter now becomes 3. The `for` loop also spawns 
3 `process` Goroutines and then `wg.Wait()` called in line no. 23 makes the 
`main` Goroutine to wait until the counter becomes zero. The counter is 
decremented by the call to `wg.Done` in the `process` Goroutine in line no. 13. 
Once all the 3 spawned Goroutines finish their execution, that is once 
`wg.Done()` has been called three times, the counter will become zero, and the 
main Goroutine will be unblocked.

**It is important to pass the address of `wg` in line no. 21. If the address is 
not passed, then each Goroutine will have its own copy of the `WaitGroup` and 
`main` will not be notified when they finish executing.**

This program outputs.

``` 
started Goroutine  2  
started Goroutine  0  
started Goroutine  1  
Goroutine 0 ended  
Goroutine 2 ended  
Goroutine 1 ended  
All go routines finished executing  
```

Your output might be different from mine since the order of execution of 
Goroutines can vary :).

### Worker Pool Implementation

One of the important uses of buffered channel is the implementation of [worker 
pool][5].

[5]: https://en.wikipedia.org/wiki/Thread_pool

In general, a worker pool is a collection of threads which are waiting for tasks 
to be assigned to them. Once they finish the task assigned, they make themselves 
available again for the next task.

We will implement worker pool using buffered channels. Our worker pool will 
carry out the task of finding the sum of a digits of the input number. For 
example if 234 is passed, the output would be 9 (2 + 3 + 4).  The input to the 
worker pool will be list of pseudo random integers.

The following are the core functionalities of our worker pool

  - Creation of a pool of Goroutines which listen on an input buffered channel 
    waiting for jobs to be assigned
  - Addition of jobs to the input buffered channel
  - Writing results to an output buffered channel after job completion
  - Read and print results from the output buffered channel

We will write this program step by step to make it easier to understand.

The first step will be creation of the structs representing the job and the 
result.

```go
type Job struct {  
    id       int
    randomno int
}

type Result struct {  
    job         Job
    sumofdigits int
}
```

Each `Job` struct has a `id` and a `randomno` for which the sum of the 
individual digits has to be computed.

The `Result` struct has a `job` field which is the job for which it holds the 
result (sum of individual digits) in the `sumofdigits` field.

The next step is to create the buffered channels for receiving the jobs and 
writing the output.

```go
var jobs = make(chan Job, 10)  
var results = make(chan Result, 10)  
```

Worker Goroutines listen for new tasks on the `jobs` buffered channel.  Once a 
task is complete, the result is written to the `results` buffered channel.

The `digits` function below does the actual job of finding the sum of the 
individual digits of an integer and returning it. We will add a sleep of 2 
seconds to this function just to simulate the fact that it takes some time for 
this function to calculate the result.

```go
func digits(number int) int {  
    sum := 0
    no := number
    for no != 0 {
        digit := no % 10
        sum += digit
        no /= 10
    }
    time.Sleep(2 * time.Second)
    return sum
}
```

Next we will write a function which creates a worker Goroutine.

```go
func worker(wg *sync.WaitGroup) {  
    for job := range jobs {
        output := Result{job, digits(job.randomno)}
        results <- output
    }
    wg.Done()
}
```

The above function creates a worker which reads from the `jobs` channel, creates 
a `Result` struct using the current `job` and the return value of the `digits` 
function and then writes the result to the `results` buffered channel. This 
function takes a WaitGroup `wg` as parameter on which it will call the `Done()` 
method when all `jobs` have been completed.

The `createWorkerPool` function will create a pool of worker Goroutines.

```go
func createWorkerPool(noOfWorkers int) {  
    var wg sync.WaitGroup
    for i := 0; i < noOfWorkers; i++ {
        wg.Add(1)
        go worker(&wg)
    }
    wg.Wait()
    close(results)
}
```

The function above takes the number of workers to be created as a parameter. It 
calls `wg.Add(1)` before creating the Goroutine to increment the WaitGroup 
counter. Then it creates the worker Goroutines by passing the address of the 
WaitGroup `wg` to the `worker` function.  After creating the needed worker 
Goroutines, it waits for all the Goroutines to finish their execution by calling 
`wg.Wait()`. After all Goroutines finish executing, it closes the `results` 
channel since all Goroutines have finished their execution and no one else will 
further be writing to the `results` channel.

Now that we have the worker pool ready, lets go ahead and write the function 
which will allocate jobs to the workers.

```go
func allocate(noOfJobs int) {  
    for i := 0; i < noOfJobs; i++ {
        randomno := rand.Intn(999)
        job := Job{i, randomno}
        jobs <- job
    }
    close(jobs)
}
```

The `allocate` function above takes the number of jobs to be created as input 
parameter, generates pseudo random numbers with a maximum value of `998`, 
creates `Job` struct using the random number and the for loop counter `i` as the 
id and then writes them to the `jobs` channel. It closes the `jobs` channel 
after writing all jobs.

Next step would be to create the function that reads the `results` channel and 
prints the output.

```go
func result(done chan bool) {  
    for result := range results {
        fmt.Printf("Job id %d, input random no %d , sum of digits %d\n", result.job.id, result.job.randomno, result.sumofdigits)
    }
    done <- true
}
```

The `result` function reads the `results` channel and prints the job id, input 
random no and the sum of digits of the random no. The result function also takes 
a `done` channel as parameter to which it writes to once it has printed all the 
results.

We have everything set now. Lets go ahead and finish the last step of calling 
all these functions from the `main()` function.

```go
func main() {  
    startTime := time.Now()
    noOfJobs := 100
    go allocate(noOfJobs)
    done := make(chan bool)
    go result(done)
    noOfWorkers := 10
    createWorkerPool(noOfWorkers)
    <-done
    endTime := time.Now()
    diff := endTime.Sub(startTime)
    fmt.Println("total time taken ", diff.Seconds(), "seconds")
}
```

We first store the execution start time of the program in line no.2 of the main 
function and in the last line (line no. 12) we calculate the time difference 
between the endTime and startTime and display the total time it took for the 
program to run. This is needed because we will do some benchmarks by changing 
the number of Goroutines.

The `noOfJobs` is set to 100 and then `allocate` is called to add jobs to the 
`jobs` channel.

Then `done` channel is created and passed to the `result` Goroutine so that it 
can start printing the output and notify once everything has been printed.

Finally a pool of `10` worker Goroutines are created by the call to 
`createWorkerPool` function and then main waits on the `done` channel for all 
the results to be printed.

Here is the full program for your reference. I have imported the necessary 
packages too.

```go
package main

import (  
    "fmt"
    "math/rand"
    "sync"
    "time"
)

type Job struct {  
    id       int
    randomno int
}
type Result struct {  
    job         Job
    sumofdigits int
}

var jobs = make(chan Job, 10)  
var results = make(chan Result, 10)

func digits(number int) int {  
    sum := 0
    no := number
    for no != 0 {
        digit := no % 10
        sum += digit
        no /= 10
    }
    time.Sleep(2 * time.Second)
    return sum
}
func worker(wg *sync.WaitGroup) {  
    for job := range jobs {
        output := Result{job, digits(job.randomno)}
        results <- output
    }
    wg.Done()
}
func createWorkerPool(noOfWorkers int) {  
    var wg sync.WaitGroup
    for i := 0; i < noOfWorkers; i++ {
        wg.Add(1)
        go worker(&wg)
    }
    wg.Wait()
    close(results)
}
func allocate(noOfJobs int) {  
    for i := 0; i < noOfJobs; i++ {
        randomno := rand.Intn(999)
        job := Job{i, randomno}
        jobs <- job
    }
    close(jobs)
}
func result(done chan bool) {  
    for result := range results {
        fmt.Printf("Job id %d, input random no %d , sum of digits %d\n", result.job.id, result.job.randomno, result.sumofdigits)
    }
    done <- true
}
func main() {  
    startTime := time.Now()
    noOfJobs := 100
    go allocate(noOfJobs)
    done := make(chan bool)
    go result(done)
    noOfWorkers := 10
    createWorkerPool(noOfWorkers)
    <-done
    endTime := time.Now()
    diff := endTime.Sub(startTime)
    fmt.Println("total time taken ", diff.Seconds(), "seconds")
}
```

[Run in playground](https://play.golang.org/p/au5islUIbx)

Please run this program in your local machine for more accuracy in the total 
time taken calculation.

This program will print,

``` 
Job id 1, input random no 636, sum of digits 15  
Job id 0, input random no 878, sum of digits 23  
Job id 9, input random no 150, sum of digits 6  
...
total time taken  20.01081009 seconds  
```

A total of 100 lines will be printed corresponding to the 100 jobs and then 
finally the total time taken for the program to run will be printed in the last 
line. Your output will differ from mine as the Goroutines can run in any order 
and the total time will also vary based on the hardware. In my case it takes 
approximately 20 seconds for the program to complete.

Now lets increase the `noOfWorkers` in the `main` function to `20`. We have 
doubled the number the workers. Since the worker Goroutines have 
increased(doubled to be precise), the total time taken for the program to 
complete should reduce(by half to be precise). In my case it became, 
10.004364685 seconds and the program printed,

``` 
...
total time taken  10.004364685 seconds  
```

Now we can understand that as the number of worker Goroutines increase, the 
total time taken to complete the jobs decreases. I leave it as an exercise for 
you to play with the `noOfJobs` and `noOfWorkers` in the `main` function to 
different values and analyse the results.

This brings us to an end of this tutorial. Have a good day.
