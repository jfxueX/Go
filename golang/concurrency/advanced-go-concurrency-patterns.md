# Advanced Go Concurrency Patterns

original: <http://blog.newbmiao.com/2018/02/09/advanced-go-concurrency-patterns.html>


* [Concurrency Is Not Parallelism](#concurrency-is-not-parallelism)
    * [Concurrency](#concurrency)
    * [Parallelism](#parallelism)
    * [VS](#vs)
    * [Go Concurrency Support](#go-concurrency-support)
* [Go Concurrency Patterns](#go-concurrency-patterns)
    * [Service Channel](#service-channel)
    * [Multiplexing](#multiplexing)
    * [Sequencing](#sequencing)
    * [for-select](#for-select)
    * [timeout-select](#timeout-select)
    * [quit channel](#quit-channel)
    * [Daisy-chain](#daisy-chain)
* [Advanced Go Concurrency Patterns](#advanced-go-concurrency-patterns)
    * [Three techniques](#three-techniques)
    * [other improvement](#other-improvement)


Go 的并发模式是一个很有意思的东西，这里先做个搬运工。本文是 Go 官方 talk 里对并
发的定义及并发的模式的一些搜集

> 原文引自：  
> Advanced Go Concurrency Patterns
> 
>   - [Youtube](https://www.youtube.com/watch?v=QDDwwePbDtw)
>   - [slide](https://talks.golang.org/2013/advconc.slide#1)
>   - [code](https://github.com/golang/talks/tree/master/2013/advconc)
> 
> Go Concurrency Patterns
> 
>   - [Youtube](https://www.youtube.com/watch?v=f6kdp27TYZs)
>   - [slide](https://talks.golang.org/2012/concurrency.slide#1)
> 
> Concurrency Is Not
>     Parallelism
> 
>   - [Youtube](https://www.youtube.com/watch?v=cN_DpYBzKso)
>   - [slide](https://talks.golang.org/2012/waza.slide#1)
>   - [并发不是并行](http://tonybai.com/2015/06/23/concurrency-and-parallelism/)

## Concurrency Is Not Parallelism

Concurrency (并发) 是程序能**组织**执行过程使同时可以处理多件事。  
Parallelism (并行) 是程序能同时**执行**多件事。

所以 Rob Pike 说：*并发关乎结构，并行关乎执行*。

### Concurrency

Concurrency is the composition of independently executing computations.  
并发是独立执行计算的组合。

Concurrency is a way to structure software, particularly as a way to write clean 
code that interacts well with the real world.  
并发是一种构建软件的方法，特别是作为编写与现实世界良好交互的干净代码的一种方式。

Concurrency is about dealing with lots of things at once.  
并发是指同时处理大量事物。

### Parallelism

Parallelism is the simultaneous execution of (possibly related) computations.  
并行是（可能相关的）计算的同时执行。

Parallelism is about doing lots of things at once.  
并行是指同时做很多事情。

### VS

Concurrency is about structure, parallelism is about execution.  
并发关乎结构，并行关乎执行。

Concurrency provides a way to structure a solution to solve a problem that may 
(but not necessarily) be parallelizable.  
并发提供了一种构建解决方案的方法，以解决可能（但不一定）可并行化的问题。

### Go Concurrency Support

  - concurrent execution (goroutines) 协程支持

  - synchronization and messaging (channels) CSP 的通信方式来共享内存，此处需了解
    [go的内存模型][1]

  - multi-way concurrent control (select) 控制协程切换
      - All channels are evaluated.  
        每个信道都会被评估
      - Selection blocks until one communication can proceed, which then does.  
        没有可处理的信道时，select 会一直阻塞
      - If multiple can proceed, select chooses pseudo-randomly.  
        多个信道可执行时，select 会伪随机选一个
      - A default clause, if present, executes immediately if no channel is ready.  
        有 default 申明时，若没有可处理信道，则 default 立马执行

[1]: http://blog.newbmiao.com/2018/02/06/go-memory-model.html

## Go Concurrency Patterns

### Service Channel

Channel 是 Go 里 `first class` 值，像 string 这些类型一样。  
用作函数返回时，通过返回的 channel 进行交互，可以起到服务一样的效果。

[运行](https://play.golang.org/p/YLBW2G3SeFp)  

```go
func main() {
    joe := boring("Joe")
    ann := boring("Ann")
    for i := 0; i < 5; i++ {
        fmt.Println(<-joe)
        fmt.Println(<-ann)
    }
    fmt.Println("You're both boring; I'm leaving.")
}

func boring(msg string) <-chan string {     // Returns receive-only channel of strings.
    c := make(chan string)
    go func() {     // We launch the goroutine from inside the function.
        for i := 0; ; i++ {
            c <- fmt.Sprintf("%s %d", msg, i)
            time.Sleep(time.Duration(rand.Intn(1e3)) * time.Millisecond)
        }
    }()
    return c    // Return the channel to the caller.
}
```

### Multiplexing

就是将多个输入处理成一个，常见实现是使用 go 启动多个协程去合并；或者使用 for-
select 去合并

[运行](https://play.golang.org/p/G4gpS8-g36Y)  

```go
func main() {
    c := fanIn(boring("Joe"), boring("Ann"))
    for i := 0; i < 10; i++ {
        fmt.Println(<-c)
    }
    fmt.Println("You're both boring; I'm leaving.")
}

func fanIn(input1, input2 <-chan string) <-chan string {
    c := make(chan string)
    go func() { for { c <- <-input1} }()
    go func() { for { c <- <-input2} }()
    return c
}
```

> - `fan-in`:  
>    A function can read from multiple inputs and proceed until all are closed by 
>    multiplexing the input channels onto a single channel that’s closed when all 
>    the inputs are closed.  
> - `fan-out`:  
>    Multiple functions can read from the same channel until that channel is 
>    closed)  
>    多个函数可以从同一信道读取，直到该信道关闭。  
> -  [pipeline](https://blog.golang.org/pipelines)

### Sequencing

按顺序执行，用**全局信道**去发送接受来控制任务依次执行  

[运行](https://play.golang.org/p/VwVSQ_4I2ex)


```go
type Message struct {
    str string
    wait chan bool
}

// main
for i := 0; i < 5; i++ {
    msg1 := <-c; fmt.Println(msg1.str)
    msg2 := <-c; fmt.Println(msg2.str)
    msg1.wait <- true
    msg2.wait <- true
}

// boring
waitForIt := make(chan bool)    // Give main control over our execution.
go func() {             // Launch the goroutine from inside the function. Function Literal.
    for i := 0; ; i++ {
        c <- Message{fmt.Sprintf("%s %d", msg, i), waitForIt}
        time.Sleep(time.Duration(rand.Intn(1e3)) * time.Millisecond)
        <-waitForIt     // Block until main tells us to go again.
    }
}()
```

### for-select

简化 go 创建多个协程的声明方式

```go
func fanIn(input1, input2 <-chan string) <-chan string {
    c := make(chan string)
    go func() {
        for {
            select {
            case s := <-input1:  c <- s
            case s := <-input2:  c <- s
            }
        }
    }()
    return c
}
```

### timeout-select

对 select 可以增加超时信道，超时则返回，避免 select 一直阻塞

```go
func main() {
    c := boring("Joe")
    for {
        select {
        case s := <-c:
            fmt.Println(s)
        case <-time.After(1 * time.Second):
            fmt.Println("You're too slow.")
            return
        }
    }
}
```

### quit channel

使用通道控制执行是否提前结束

  - just quit  
    ```go
    // main
    quit := make(chan struct{})
    c := boring("Joe", quit)
    for i := rand.Intn(10); i >= 0; i-- { 
        fmt.Println(<-c) 
    }
    quit <- struct{}
        select {
        case c <- fmt.Sprintf("%s: %d", msg, i):
            // do nothing
        case <-quit:
            return
        }
    ```
  - use quit deliver msg  
    ```go
    quit := make(chan string)
    ```

### Daisy-chain

关于这个模式 stackoverflow 有个[讨论][2]，可以看看

[2]: https://stackoverflow.com/questions/26135616/understand-the-code-go-concurrency-pattern-daisy-chain

```go
func f(left, right chan int) {
    left <- 1 + <-right
}

func main() {
    const n = 10000
    leftmost := make(chan int)
    right := leftmost
    left := leftmost
    for i := 0; i < n; i++ {
        right = make(chan int)
        go f(left, right)
        left = right
    }
    go func(c chan int) { c <- 1 }(right)
    fmt.Println(<-leftmost)
}
```


## Advanced Go Concurrency Patterns

### Three techniques

  - for-select loop  
    select 防止 loop 阻塞在某一状态，便于调度多项任务

  - service channel, reply channels (chan chan error)  
    service channel 就是常见的 pattern，不多说  
    reply channels 实现了 close 操作中关闭和错误返回无 `data race`:  
    **close 通过 `chan chan error` 向 loop 请求关闭，并等待其返回关闭前是否有错误**  
    ```go
    func (s *sub) Close() error {
        errc := make(chan error)
        s.closing <- errc // HLchan  // 请求关闭
        return <-errc     // HLchan  // 等待结果返回
    }
    // in loop
    var err error
    for{
        select{
            case errc := <-s.closing: // 收到关闭请求
                errc <- err           // 返回错误
                close(s.updates)      // 执行关闭
                return
        }
    }
    ```

  - nil channels in select cases  
    向值为 nil 的 channel 发送和接收都会阻塞，在 select 中使用它可控制是否执行

此外，分享中提到一些点也值得注意：

### other improvement

  - limit：限制 pending （处理任务队列）的数目，不要让其无限增大，避免过多请求及
    内存消耗

  - async io：拆解 fetch 的请求和结果处理，使用状态 channel 同步，使 fetch 中不
    要阻塞其他处理进行  
    **拆解前**  
    ```go
    case <-startFetch:
        var fetched []Item
        fetched, next, err = s.fetcher.Fetch()
        if err != nil {
            next = time.Now().Add(10 * time.Second)
            break
        }
        for _, item := range fetched {
            if !seen[item.GUID] {
                pending = append(pending, item)
                seen[item.GUID] = true         
            }
        }
    ```

    **拆解后**  
    startFetch 和 fetchDone 同一时刻，只有一个不为 `nil`  
    ```go
    type fetchResult struct{ fetched []Item; next time.Time; err error }
        var fetchDone chan fetchResult // if non-nil, Fetch is running
            var startFetch <-chan time.Time
            if fetchDone == nil && len(pending) < maxPending {
                startFetch = time.After(fetchDelay) // enable fetch case
            }
            select {
            case <-startFetch:
                fetchDone = make(chan fetchResult, 1)
                go func() {
                    fetched, next, err := s.fetcher.Fetch()
                    fetchDone <- fetchResult{fetched, next, err}
                }()
            case result := <-fetchDone: 
                fetchDone = nil
                // Use result.fetched, result.next, result.err
    ```

这是对以上模式整合实现的一个 rss 聚合器 demo，可以仔细研究下  
[运行](https://play.golang.org/p/j_npc1o3zZp)

