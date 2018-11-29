# Chapter 2. Understanding the Concurrency Model

Now that we have a sense of what Go is capable of and how to test drive some 
concurrency models, we need to look deeper into Go's most powerful features to 
understand how to best utilize various concurrent tools and models.

We played with some general and basic goroutines to see how we can run
concurrent processes, but we need to see how Go manages scheduling in
concurrency before we get to communication between channels.

By this point, you should be well-versed in what goroutines do, but it's worth 
understanding *how* they work internally in Go. Go handles concurrency with 
cooperative scheduling, which, as we mentioned in the previous chapter, is 
heavily dependent on some form of blocking code.

The most common alternative to cooperative scheduling is preemptive scheduling, 
wherein each subprocess is granted a space of time to complete and then its 
execution is paused for the next.

Without some form of yielding back to the main thread, execution runs into 
issues. This is because Go works with a single process, working as a conductor 
for an orchestra of goroutines. Each subprocess is responsible to announce its 
own completion. As compared to other concurrency models, some of which allow for 
direct, named communication, this might pose a sticking point, particularly if 
you haven't worked with channels before.

You can probably see a potential for deadlocks given these facts. In this 
chapter, we'll discuss both the ways Go's design allows us to manage this and 
the methods to mitigate issues in applications wherein it fails.


## Visualizing concurrency

Our first attempt at visualizing concurrency will have two simple goroutines 
running the `drawPoint` function in a loop with 100 iterations. After running 
this, you can visit `localhost:1900/visualize` and see what concurrent 
goroutines look like.

If you run into problems with port 1900 (either with your firewall or through a 
port conflict), feel free to change the value on line 99 in the `main()` 
function. You may also need to access it through `127.0.0.1` if your system 
doesn't resolve localhost.

Note that we're not using `WaitGroup` or anything to manage the end of the 
goroutines because all we want to see is a visual representation of our code 
running. You can also handle this with a specific blocking code or 
`runtime.Gosched()`, as shown:

```go
package main

import (
    "github.com/ajstarks/svgo"
    "net/http"
    "fmt"
    "log"
    "time"
    "strconv"
)

var width = 800
var height = 400
var startTime = time.Now().UnixNano()


func drawPoint(osvg *svg.SVG, pnt int, process int) {
    sec := time.Now().UnixNano()
    diff := ( int64(sec) - int64(startTime) ) / 100000

    pointLocation := 0
    
    pointLocation = int(diff)
    pointLocationV := 0
    color := "#000000"
    switch {
    case process == 1:
        pointLocationV = 60
        color = "#cc6666"
    default:
        pointLocationV = 180
        color = "#66cc66"
    }

    osvg.Rect(pointLocation,pointLocationV,3,5,"fill:"+color+";stroke:
    none;")
    time.Sleep(150 * time.Millisecond)
}

func visualize(rw http.ResponseWriter, req *http.Request) {
    startTime = time.Now().UnixNano()
    fmt.Println("Request to /visualize")
    rw.Header().Set("Content-Type", "image/svg+xml")
    
    outputSVG := svg.New(rw)

    outputSVG.Start(width, height)
    outputSVG.Rect(10, 10, 780, 100, "fill:#eeeeee;stroke:none")
    outputSVG.Text(20, 30, "Process 1 Timeline", "text-
        anchor:start;font-size:12px;fill:#333333")
    outputSVG.Rect(10, 130, 780, 100, "fill:#eeeeee;stroke:none")    
    outputSVG.Text(20, 150, "Process 2 Timeline", "text-
        anchor:start;font-size:12px;fill:#333333")  

    for i:= 0; i < 801; i++ {
        timeText := strconv.FormatInt(int64(i),10)
        if i % 100 == 0 {
            outputSVG.Text(i,380,timeText,"text-anchor:middle;font-size:10px;fill:#000000")      
        }else if i % 4 == 0 {
          outputSVG.Circle(i,377,1,"fill:#cccccc;stroke:none")  
        }

        if i % 10 == 0 {
            outputSVG.Rect(i,0,1,400,"fill:#dddddd")
        }
        if i % 50 == 0 {
            outputSVG.Rect(i,0,1,400,"fill:#cccccc")
        }
    }

    for i := 0; i < 100; i++ {
        go drawPoint(outputSVG,i,1)
        drawPoint(outputSVG,i,2)    
    }

    outputSVG.Text(650, 360, "Run without goroutines", "text-anchor:start;font-size:12px;fill:#333333")      
    outputSVG.End()
}

func main() {
  http.Handle("/visualize", http.HandlerFunc(visualize))

    err := http.ListenAndServe(":1900", nil)
    if err != nil {
        log.Fatal("ListenAndServe:", err)
    }
```

When you go to `localhost:1900/visualize`, you should see something like the 
following screenshot:

![Visualizing
concurrency](/library/view/mastering-concurrency-in/9781783983483/graphics/3483OS_02_03.jpg)

As you can see, everything is definitely running concurrently—our briefly 
sleeping goroutines hit on the timeline at the same moment. By simply forcing 
the goroutines to run in a serial fashion, you'll see a predictable change in 
this behavior.  Remove the goroutine call on line 73, as shown:

```go
    drawPoint(outputSVG,i,1)
    drawPoint(outputSVG,i,2)  
```

To keep our demonstration clean, change line 77 to indicate that there are no 
goroutines as follows:

```go
outputSVG.Text(650, 360, "Run with goroutines", "text-anchor:start;font-size:12px;fill:#333333")  
```

If we stop our server and restart with `go run`, we should see something like 
the following screenshot:

![Visualizing
concurrency](/library/view/mastering-concurrency-in/9781783983483/graphics/3483OS_02_04.jpg)

Now, each process waits for the previous process to complete before beginning. 
You can actually add this sort of feedback to any application if you run into 
problems with syncing data, channels, and processes.

If we so desired, we could add some channels and show communication across them 
as represented. Later, we will design a self-diagnosing server that gives real-
time analytics about the state and status of the server, requests, and channels.

If we turn the goroutine back on and increase our maximum available processors, 
we'll see something similar to the following screenshot, which is not exactly 
the same as our first screenshot:

![Visualizing
concurrency](/library/view/mastering-concurrency-in/9781783983483/graphics/3483OS_02_05.jpg)

Your mileage will obviously vary depending on server speeds, the number of 
processors, and so on.  But in this case, our change here resulted in a faster 
total execution time for our two processes with intermittent sleeps. This should 
come as no surprise, given we have essentially twice the bandwidth available to 
complete the two tasks.


## RSS in action

Let's take the concept of **Rich Site Summary** / **Really Simple Syndication** 
(**RSS**) and inject some real potential delays to identify where we can best 
utilize goroutines in an effort to speed up execution and prevent blocking code. 
One common way to bring real-life, potentially blocking application elements 
into your code is to use something involving network transmission.

This is also a great place to look at timeouts and close channels to ensure that 
our program doesn't fall apart if something takes too long.

To accomplish both these requirements, we'll build a very basic RSS reader that 
will simply parse through and grab the contents of five RSS feeds. We'll read 
each of these as well as the provided links on each, and then we'll generate an 
SVG report of the process available via HTTP.

### Note

This is obviously an application best suited for a background task—you'll notice 
that each request can take a long time. However, for graphically representing a 
real-life process working with and without concurrency, it will work, especially 
with a single end user.  We'll also log our steps to standard output, so be sure 
to take a look at your console as well.

For this example, we'll again use a third-party library, although it's entirely 
possible to parse RSS using Go's built-in XML package. Given the open-ended 
nature of XML and the specificity of RSS, we'll bypass them and use `go-pkg-rss` 
by Jim Teeuwen, available via the following `go get` command:

```bash
go get github.com/jteeuwen/go-pkg-rss
```

While this package is specifically intended as a replacement for the Google 
Reader product, which means that it does interval-based polling for new content 
within a set collection of sources, it also has a fairly neat and tidy RSS 
reading implementation. There are a few other RSS parsing libraries out there, 
though, so feel free to experiment.

## An RSS reader with self diagnostics

Let's take a look at what we've learned so far, and use it to fetch and parse a 
set of RSS feeds concurrently while returning some visual feedback about the 
process in an internal web browser, as shown in the following code:

```go
package main

import(
  "github.com/ajstarks/svgo"
  rss "github.com/jteeuwen/go-pkg-rss"    
  "net/http"
  "log"
  "fmt"
  "strconv"
  "time"
  "os"
  "sync"
  "runtime"
)

type Feed struct {
  url string
  status int
  itemCount int
  complete bool
  itemsComplete bool
  index int
}
```

Here is the basis of our feed's overall structure: we have a `url` variable that 
represents the feed's location, a `status` variable to indicate whether it's 
started, and a `complete` Boolean variable to indicate it's finished. The next 
piece is an individual `FeedItem`; here's how it can be laid out:

```go
type FeedItem struct {
  feedIndex int
  complete bool
  url string
}
```

Meanwhile, we will not do much with individual items; at this point, we simply 
maintain a URL, whether it's complete or a `FeedItem` struct's index.

```go
var feeds []Feed
var height int
var width int
var colors []string
var startTime int64
var timeout int
var feedSpace int

var wg sync.WaitGroup

func grabFeed(feed *Feed, feedChan chan bool, osvg *svg.SVG) {
    startGrab := time.Now().Unix()
    startGrabSeconds := startGrab - startTime

    fmt.Println("Grabbing feed",feed.url,"at",startGrabSeconds,"second mark")

    if feed.status == 0 {
        fmt.Println("Feed not yet read")
        feed.status = 1

        startX := int(startGrabSeconds * 33);
        startY := feedSpace * (feed.index)

        fmt.Println(startY)
        wg.Add(1)

        rssFeed := rss.New(timeout, true, channelHandler, itemsHandler);

        if err := rssFeed.Fetch(feed.url, nil); err != nil {
            fmt.Fprintf(os.Stderr, "[e] %s: %s", feed.url, err)
            return
        } else {
            endSec := time.Now().Unix()    
            endX := int( (endSec - startGrab) )
            if endX == 0 {
                endX = 1
            }
            fmt.Println("Read feed in",endX,"seconds")
            osvg.Rect(startX,startY,endX,feedSpace,"fill:#000000;opacity:.4")
            wg.Wait()
            
            endGrab := time.Now().Unix()
            endGrabSeconds := endGrab - startTime
            feedEndX := int(endGrabSeconds * 33);      

            osvg.Rect(feedEndX,startY,1,feedSpace,"fill:#ff0000;opacity:.9")

            feedChan <- true
        }

    } else if feed.status == 1 {
      fmt.Println("Feed already in progress")
    }
}
```

The `grabFeed()` method directly controls the flow of grabbing any individual 
feed. It also bypasses potential concurrent duplication through the `WaitGroup` 
struct. Next, let's check out the `itemsHandler` function:

```go
func channelHandler(feed *rss.Feed, newchannels []*rss.Channel) {

}

func itemsHandler(feed *rss.Feed, ch *rss.Channel, newitems []*rss.Item) {

    fmt.Println("Found",len(newitems),"items in",feed.Url)

    for i := range newitems {
        url := *newitems[i].Guid
        fmt.Println(url)
    }

    wg.Done()
}
```

The `itemsHandler` function doesn't do much at this point, other than 
instantiating a new `FeedItem` struct—in the real world, we'd take this as the 
next step and retrieve the values of the items themselves. Our next step is to 
look at the process that grabs individual feeds and marks the time taken for 
each one, as follows:

```go
func getRSS(rw http.ResponseWriter, req *http.Request) {
    startTime = time.Now().Unix()  
    rw.Header().Set("Content-Type", "image/svg+xml")
    outputSVG := svg.New(rw)
    outputSVG.Start(width, height)

    feedSpace = (height-20) / len(feeds)

    for i:= 0; i < 30000; i++ {
        timeText := strconv.FormatInt(int64(i/10),10)
        if i % 1000 == 0 {
            outputSVG.Text(i/30,390,timeText,"text-anchor:middle;font-size:10px;fill:#000000")      
        } else if i % 4 == 0 {
            outputSVG.Circle(i,377,1,"fill:#cccccc;stroke:none")  
        }

        if i % 10 == 0 {
            outputSVG.Rect(i,0,1,400,"fill:#dddddd")
        }
        if i % 50 == 0 {
            outputSVG.Rect(i,0,1,400,"fill:#cccccc")
        }
    }

    feedChan := make(chan bool, 3)

    for i := range feeds {
        outputSVG.Rect(0, (i*feedSpace), width, feedSpace, "fill:"+colors[i]+";stroke:none;")
        feeds[i].status = 0
        go grabFeed(&feeds[i], feedChan, outputSVG)
        <- feedChan
    }

    outputSVG.End()
}
```

Here, we retrieve the RSS feed and mark points on our SVG with the status of our 
retrieval and read events. Our `main()` function will primarily handle the setup 
of feeds, as follows:

```go
func main() {
    runtime.GOMAXPROCS(2)

    timeout = 1000

    width = 1000
    height = 400

    feeds = append(feeds, Feed{index: 0, url: 
        "https://groups.google.com/forum/feed/golang-nuts/msgs/rss_v2_0.xml?num=50", 
        status: 0, itemCount: 0, complete: false, itemsComplete: false})
    feeds = append(feeds, Feed{index: 1, url: 
        "http://www.reddit.com/r/golang/.rss", status: 0, 
        itemCount: 0, complete: false, itemsComplete: false})
    feeds = append(feeds, Feed{index: 2, url: 
        "https://groups.google.com/forum/feed/golang-dev/msgs/rss_v2_0.xml?num=50", 
        status: 0, itemCount: 0, complete: false, itemsComplete: false })
```

Here is our slice of `FeedItem` structs:

```go
  colors = append(colors,"#ff9999")
  colors = append(colors,"#99ff99")
  colors = append(colors,"#9999ff")  
```

In the print version, these colors may not be particularly useful, but testing 
it on your system will allow you to delineate between events inside the 
application. We'll need an HTTP route to act as an endpoint; here's how we'll 
set that up:

```go
http.Handle("/getrss", http.HandlerFunc(getRSS))
    err := http.ListenAndServe(":1900", nil)
    if err != nil {
        log.Fatal("ListenAndServe:", err)
    }  
}
```

When run, you should see the start and duration of the RSS feed retrieval and 
parsing, followed by a thin line indicating that the feed has been parsed and 
all items read.

Each of the three blocks expresses the full time to process each feed, 
demonstrating the nonconcurrent execution of this version, as shown in the 
following screenshot:

Note that we don't do anything interesting with the feed items, we simply read 
the URL. The next step will be to grab the items via HTTP, as shown in the 
following code snippet:

```go
url := *newitems[i].Guid
    response, _, err := http.Get(url)
    if err != nil {

    }
```

With this example, we stop at every step to provide some sort of feedback to the 
SVG that some event has occurred. Our channel here is buffered and we explicitly 
state that it must receive three Boolean messages before it can finish blocking, 
as shown in the following code snippet:

```go
feedChan := make(chan bool, 3)

for i := range feeds {
    outputSVG.Rect(0, (i*feedSpace), width, feedSpace, "fill:"+colors[i]+";stroke:none;")
    feeds[i].status = 0
    go grabFeed(&feeds[i], feedChan, outputSVG)
    <- feedChan
}

outputSVG.End()
```

By giving `3` as the second parameter in our channel invocation, we tell Go that 
this channel must receive three responses before continuing the application. You 
should use caution with this, though, particularly in setting things explicitly 
as we have done here. What if one of the goroutines never sent a Boolean across 
the channel? The application would crash.

Note that we also increased our timeline here, from 800ms to 60 seconds, to 
allow for retrieval of all feeds. Keep in mind that if our script exceeds 60 
seconds, all actions beyond that time will occur outside of this visual timeline 
representation.

By implementing the `WaitGroup` struct while reading feeds, we impose some 
serialization and synchronization to the application. The second feed will not 
start until the first feed has completed retrieving all URLs. You can probably 
see where this might introduce some errors going forward:

```go
wg.Add(1)
rssFeed := rss.New(timeout, true, channelHandler, itemsHandler);
…
wg.Wait()
```

This tells our application to yield until we set the `Done()` command from the 
`itemsHandler()` function.

So what happens if we remove `WaitGroups` entirely? Given that the calls to grab 
the feed items are asynchronous, we may not see the status of all of our RSS 
calls; instead, we might see just one or two feeds or no feed at all.

## Imposing a timeout

So what happens if nothing runs within our timeline? As you might expect, we'll 
get three bars with no activity in them. It's important to consider how to kill 
processes that aren't doing what we expect them to. In this case, the best 
method is a timeout. The `Get` method in the `http` package does not natively 
support a timeout, so you'll have to roll your own `rssFeed.Fetch` (and 
underlying `http.Get()`) implementation if you want to prevent these requests 
from going into perpetuity and killing your application. We'll dig into this a 
bit later; in the mean time, take a look at the `Transport` struct, available in 
the core `http` package at <http://golang.org/pkg/net/http/#Transport>.


## A little bit about CSP

We touched on CSP briefly in the previous chapter, but it's worth exploring a 
bit more in the context of how Go's concurrency model operates.

CSP evolved in the late 1970s and early 1980s through the work of Sir Tony Hoare 
and is still in the midst of evolution today. Go's implementation is heavily 
based on CSP, but it neither entirely follows all the rules and conventions set 
forth in its initial description nor does it follow its evolution since.

One of the ways in which Go differs from true CSP is that as it is defined, a 
process in Go will only continue so long as there exists a channel ready to 
receive from that process. We've already encountered a couple of deadlocks that 
were the result of a listening channel with nothing to receive. The inverse is 
also true; a deadlock can result from a channel continuing without sending 
anything, leaving its receiving channel hanging indefinitely.

This behavior is endemic to Go's scheduler, and it should really only pose 
problems when you're working with channels initially.

### Note

Hoare's original work is now available (mostly) free from a number of 
institutions. You can read, cite, copy, and redistribute it free of charge (but 
not for commercial gain). If you want to read the whole thing, you can grab it 
at <http://www.cs.ucf.edu/courses/cop4020/sum2009/CSP-hoare.pdf>.

The complete book itself is also available at <http://www.usingcsp.com/
cspbook.pdf>.

As of this publishing, Hoare is working as a researcher at Microsoft.

As per the designers of the application itself, the goal of Go's implementation 
of CSP concepts was to focus on simplicity—you don't have to worry about threads 
or mutexes unless you really want to or need to.

## The dining philosophers problem

You may have heard of the dining philosophers problem, which describes the kind 
of problems concurrent programming was designed to solve. The dining 
philosophers problem was formulated by the great Edsger Dijkstra. The crux of 
the problem is a matter of resources—five philosophers sit at a table with five 
plates of food and five forks, and each can only eat when he has two forks (one 
to his left and another to his right). A visual representation is shown as 
follows:

![The dining philosophers
problem](/library/view/mastering-concurrency-in/9781783983483/graphics/3483OS_02_07.jpg)

With a single fork on either side, any given philosopher can only eat when he 
has a fork in both hands and must put both back on the table when complete.  The 
idea is to coordinate the meal such that all of the philosophers can eat in 
perpetuity without any starving—two philosophers must be able to eat at any 
moment and there can be no deadlocks. They're philosophers because when they're 
not eating, they're thinking. In a programming analog, you can consider this as 
either a waiting channel or a sleeping process.

Go handles this problem pretty succinctly with goroutines. Given five 
philosophers (in an individual struct, for example), you can have all five 
alternate between thinking, receiving a notification when the forks are down, 
grabbing forks, dining with forks, and placing the forks down.

Receiving the notification that the forks are down acts as the listening 
channel, dining and thinking are separate processes, and placing the forks down 
operates as an announcement along the channel.

We can visualize this concept in the following pseudo Go code:

```go
type Philosopher struct {
    leftHand bool
    rightHand bool
    status int
    name string
}

func main() {
    philosophers := [...]Philospher{"Kant", "Turing", "Descartes","Kierkegaard","Wittgenstein"}

    evaluate := func() {
        for {
            select {
                case <- forkUp:
                    // philosophers think!
                case <- forkDown:
                    // next philospher eats in round robin
            }
        }
    }
}
```

This example has been left very abstract and nonoperational so that you have a 
chance to attempt to solve it. We will build a functional solution for this in 
the next chapter, so make sure to compare your solution later on.

There are hundreds of ways to handle this problem, and we'll look at a couple of 
alternatives and how they can or cannot play nicely within Go itself.


## Go and the actor model

The actor model is something that you'll likely be very familiar with if you're 
an Erlang or Scala user. The difference between CSP and the actor model is 
negligible but important.  With CSP, messages from one channel can only be 
completely sent if another channel is listening and ready for them. The actor 
model does not necessarily require a ready channel for another to send. In fact, 
it stresses direct communication rather than relying on the conduit of a 
channel.

Both systems can be nondeterministic, which we've already seen demonstrated in 
Go/CSP in our earlier examples. CSP and goroutines are anonymous and 
transmission is specified by the channel rather than the source and destination. 
An easy way to visualize this in pseudocode in the actor model is as follows:

```go
a = new Actor
b = new Actor
a -> b("message")
```

In CSP, it is as follows:

```go
a = new Actor
b = new Actor
c = new Channel
a -> c("sending something")
b <- c("receiving something")
```

Both serve the same fundamental functionality but through slightly different 
ways.


## Object orientation

As you work with Go, you will notice that there is a core characteristic that's 
often espoused, which users may feel is wrong. You'll hear that Go is not an 
object-oriented language, and yet you have structs that can have methods, those 
methods in turn can have methods, and you can have communication to and from any 
instantiation of it. Channels themselves may feel like primitive object 
interfaces, capable of setting and receiving values from a given data element.

The message passing implementation of Go is, indeed, a core concept of object-
oriented programming. Structs with interfaces operate essentially as classes, 
and Go supports polymorphism (although not parametric polymorphism). Yet, many 
who work with the language (and who have designed it) stress that it is not 
object oriented. So what gives?

Much of this definition ultimately depends on who you ask. Some believe that Go 
lacks some of the requisite characteristics of object-oriented programming, and 
others believe it satisfies them. The most important thing to keep in mind is 
that you're not limited by Go's design. Anything that you can do in a *true* 
object-oriented language can be handled without much struggle within Go.

## Demonstrating simple polymorphism in Go

As mentioned before, if you expect polymorphism to resemble object-oriented 
programming, this may not represent a syntactical analogue. However, the use of 
interfaces as an abstraction of class-bound polymorphic methods is just as 
clean, and in many ways, more explicit and readable. Let's look at a very simple 
implementation of polymorphism in Go:

```go
type intInterface struct {

}

type stringInterface struct {

}

func (number intInterface) Add (a int, b int) int {
  return a + b;
}

func (text stringInterface) Add (a string, b string) string {
    return a + b
}

func main() {
    number := new (intInterface)
    fmt.Println( number.Add(1, 2) )
    text := new (stringInterface)
    fmt.Println( text.Add("this old man", " he played one"))
}
```

As you can see, we use an interface (or its Go analog) to disambiguate methods. 
You cannot have generics the same way you might in Java, for example. This, 
however, boils down to a mere matter of style in the end. You should neither 
find this daunting nor will it impose any cruft or ambiguity into your code.


## Using concurrency

It hasn't yet been mentioned, but we should be aware that concurrency is not 
always necessary and beneficial for an application. There exists no real rule of 
thumb, and it's rare that concurrency will introduce problems to an application; 
but if you really think about applications as a whole, not all will require 
concurrent processes.

So what works best? As we've seen in the previous example, anything that 
introduces potential latency or I/O blocking, such as network calls, disk reads, 
third-party applications (primarily databases), and distributed systems, can 
benefit from concurrency. If you have the ability to do work while other work is 
being done on an undetermined timeline, concurrency strategies can improve the 
speed and reliability of an application.

The lesson here is you should never feel compelled to shoehorn concurrency into 
an application that doesn't really require it. Programs with inter-process 
dependencies (or lack of blocking and external dependencies) may see little or 
no benefit from implementing concurrency structures.


## Managing threads

So far, you've probably noticed that thread management is not a matter that 
requires the programmer's utmost concern in Go. This is by design. Goroutines 
aren't tied to a specific thread or threads that are handled by Go's internal 
scheduler. However, this doesn't mean that you neither have access to the 
threads nor the ability to control what individual threads do. As you know, you 
can already tell Go how many threads you have (or wish to use) by using 
`GOMAXPROCS`. We also know that using this can introduce asynchronous issues as it 
pertains to data consistency and execution order.

At this point, the main issue with threads is not how they're accessed or 
utilized, but how to properly control execution flow to guarantee that your data 
is predictable and synchronized.


## Using sync and mutexes to lock data

One issue that you may have run into with the preceding examples is the notion 
of atomic data. After all, if you deal with variables and structures across 
multiple goroutines, and possibly processors, how do you ensure that your data 
is safe across them? If these processes run in parallel, coordinating data 
access can sometimes be problematic.

Go provides a bevy of tools in its `sync` package to handle these types of 
problems. How elegantly you approach them depends heavily on your approach, but 
you should never have to reinvent the wheel in this realm.

We've already looked at the `WaitGroup` struct, which provides a simple method 
to tell the main thread to pause until the next notification that says a waiting 
process has done what it's supposed to do.

Go also provides a direct abstraction to a mutex. It may seem contradictory to 
call something a direct abstraction, but the truth is you don't have access to 
Go's scheduler, only an approximation of a true mutex.

We can use a mutex to lock and unlock data and guarantee atomicity in our data. 
In many cases, this may not be necessary; there are a great many times where the 
order of execution does not impact the consistency of the underlying data. 
However, when we do have concerns about this value, it's helpful to be able to 
invoke a lock explicitly. Let's take the following example:

```go
package main

import(
    "fmt"
    "sync"
)

func main() {
    current := 0
    iterations := 100
    wg := new(sync.WaitGroup);

    for i := 0; i < iterations; i++ {
        wg.Add(1)

        go func() {
            current++
            fmt.Println(current)
            wg.Done()
        }()
        wg.Wait()
    }
}
```

Unsurprisingly, this provides a list of 0 to 99 in your terminal. What happens 
if we change `WaitGroup` to know there will be 100 instances of `Done()` called, 
and put our blocking code at the end of the loop?

To demonstrate a simple proposition of why and how to best utilize `waitGroups` 
as a mechanism for concurrency control, let's do a simple number iterator and 
look at the results. We will also check out how a directly called mutex can 
augment this functionality, as follows:

```go
func main() {
    runtime.GOMAXPROCS(2)
    current := 0
    iterations := 100
    wg := new (sync.WaitGroup);
    wg.Add(iterations)
    for i := 0; i < iterations; i++ {
        go func() {
            current++
            fmt.Println(current)
            wg.Done()
        }()
    }
    wg.Wait()
}
```

Now, our order of execution is suddenly off. You may see something like the 
following output:

```
95
96
98
99
100
3
4
```

We have the ability to lock and unlock the current command at will; however, 
this won't change the underlying execution order, it will only prevent reading 
and/or writing to and from a variable until an unlock is called.

Let's try to lock down the variable we're outputting using `mutex`, as follows:

```go
for i := 0; i < iterations; i++ {
    go func() {
        mutex.Lock()
        fmt.Println(current)
        current++
        mutex.Unlock()
        fmt.Println(current)
        wg.Done()
    }()
}
```

You can probably see how a mutex control mechanism can be important to enforce 
data integrity in your concurrent application. We'll look more at mutexes and 
locking and unlocking processes in [Chapter 4][1], *Data Integrity in an 
Application*.

[1]: ./ch04-data-integrity-in-an-application.md

## Summary

In this chapter, we've tried to remove some of the ambiguity of Go's concurrency 
patterns and models by giving visual, real-time feedback to a few applications, 
including a rudimentary RSS aggregator and reader. We examined the dining 
philosophers problem and looked at ways you can use the Go concurrency topics to 
solve the problem neatly and succinctly. We compared the way CSP and actor 
models are similar and ways in which they differ.

In the next chapter, we will take these concepts and apply them to the process 
of developing a strategy to maintain concurrency in an application.
