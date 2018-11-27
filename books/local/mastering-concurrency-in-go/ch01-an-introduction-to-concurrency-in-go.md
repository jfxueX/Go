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
