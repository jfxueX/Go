## 1：初识 Goroutine 

[例1.1](../exmaples/concurrency/e1_1.go) 是一个简单的并发程序, 核心的逻辑是：

```go
	go outputText(hello)
	outputText(world)
```

这个程序完整的打出了对应的字符串，这是因为每次循环里面都 sleep(1ms) 了，这让后面的 
`outputText(world)` 可以比前面的 Goroutine 里面的 `go outputText(hello)` 执行的时间变长，所以主进程
会一直等待钱前面的 Goroutine.

如果把这2行代码稍微改一下，就看不到任何输出了，因为主进程直接退出了：

```go
go outputText(hello)
go outputText(world)
```

`sync` 中提供了一些同步支持，在[例1.2](../exmaples/concurrency/e1_2.go)中 `outputText` 中用
`sig.Done()`来通知`main goroutine`，当前的`goroutine`执行结束.  在 `main goroutine` 中还需要用 
`g.Add(2)` 来注册需要等待的信号数. 然后执行 `g.Wait()` `main goroutine` 就会一直等待，直到监听到 2 
个 `Done` 信号.

## 2：并发模型

Go 的并发模型基于[Communicating Sequential Processes(CSP)](Communicating sequential processes),
这是一个基于 `Channel` 通信的并发模型. 这是一个可以严谨数学论证的模型，
从数学上描述了 `Goroutine Scheduler` 的工作机制.

值得一提的还有[Actor模型](https://en.wikipedia.org/wiki/Actor_model)，
它和CSP都是并发模型，但存在关键差异：

* CSP模型只有在`Channel`的监听者`ready`时才能发消息；
`Channel`有点像打电话，如果对方不在线，就会漏接；

* Actor模型有点像发短信，收发双方可以不管对方的状态，不依赖`Channel`，
甚至可以在不同主机上；比如`Erlang`就是基于`Actor模型的`

下面是Actor和CSP的消息传递机制伪码，A向B发送个消息：

1) 在Actor模型下,

```
A = new Actor
B = new Actor
A -> B(message)
```

2) 在CSP模型下,

```go
A = new Actor
B = new Actor
C = new Channel

A -> C("sending something")
C <- C("receiving something")
```

## 3: 并发策略

Go 的并发实现是 `Goroutine`，在 `runtime` 中集成了 `goroutine scheduler`，用以调度 `goroutine` 的生
命周期.

Unix/Linux 系统下，`goroutine` 实际运行还是通过内核的多线程实现的，通过内置的 `scheduler` 来调度 
`goroutine` 到内核线程上运行.

例3.1: `concurrency/e3_1.go` 说明了 `sync.WaitGroup` 的基本用法

```go
package main

import (
    "sync"
)

func main() {
    wg := new(sync.WaitGroup)
    
    wg.Add(1)
    wg.Done()
    wg.Wait()
}
```

`wg.Add(N)` 说明了主进程要等待 N 个 `wg.Done()` 信号，一般来说 `wg.Done()` 会在 `goroutine` 里面调
用.

例3.2: `concurrency/e3_2.go`

```go
package main

import (
    "sync"
)

func main() {
    wg := new(sync.WaitGroup)
    wg.Add(1)
    
    go func(signal *sync.WaitGroup) {
        signal.Done()
    }(wg)
    
    wg.Wait()
}
```

主线程里面调用 `wg.Wait()` 实现等待，接收到 N 个 `wg.Done` 信号之后，主线程停止阻塞.

源码里面的注释说明了 `WaitGroup.Add()` 的使用，可以多次调用，输入可以是负数，只要 `counter = 0`, 阻
塞就取消了，如果 `counter < 0`，线程 panic 

```go
// Add adds delta, which may be negative, to the WaitGroup counter.
// If the counter becomes zero, all goroutines blocked on Wait are released.
// If the counter goes negative, Add panics.
```

`wg.Done()` 的实现是 `wg.Add(-1)`

```go
func (wg *WaitGroup) Done() {
    wg.Add(-1)
}
```

例3.3: `concurrency/e3_3.go`

模拟了一个转账的场景，有 N 元钱的存款，经过 M 次并发的消费

```go
func main() {
    ...
    // 等待1个Done信号
    wg.Add(1)
    for i := 0; i < 100; i++ {
        go func(ii int, trChan chan (bool)) {
            ...
            // 最后一次交易时向传入的Channel发送一个消息
            if ii == 99 {
                trChan <- true
            }
        }(i, tranChan)
    
    }

    // 主线程中监听Channel中的消息
    select {
    case <-tranChan:
    // 最后一次交易的时候，Channel会收到信息，接着这里调用了Done
    // 实现了祝线程等待N个并发的交易线程的模型
        fmt.Println("Transactions finished")
        wg.Done()
    
    }

    // 主线程中的等到点，在足够的Done信号来临之前，Wait的调用线程都会被阻塞
    wg.Wait()

    // 用完Channel记得清理干净
    close(tranChan)
}
```

这个程序存在2个比较大的问题：

* 直接创建了100个`goroutine`，在第100个`goroutine`执行的时候更新`Channel`.
  看起来好像不错，但如果99的goroutine第一个执行，也就意味着只要它执行完主线程
  就可以直接退出了

* 没有保护公有数据`balance`，在比较高的并发现可能出现数据读写不一致的情况

例3.4 `concurrency/e3_4.go` 尝试改进一下例3.3

* 去掉了不必要的`channel`，转而用`WaitGroup`实现同步，保证100个子线程都全部完成

* 加入`sync.Mutex`支持，在读写balance的时候需要保证数据的强一致性

例3.3中用`channel`不是必须的，而且还有一些额外的性能开销，存在多个子线程时，
而且在这些子线程中存在同步，可以考虑不用channel，转而用`sync.WaitGroup`来实现.

例3.5 `concurrency/e3_5.go` 用`time.After`控制`channel`的timeout时间

例3.6 `concurrency/e3_6.go` 基于`net/http/pprof`的实时监控方案

`import _ "net/http/pprof"`会创建performance相关的http接口

```go
// net/http/pprof/pprof.go#L67
func init() {
	http.Handle("/debug/pprof/", http.HandlerFunc(Index))
	http.Handle("/debug/pprof/cmdline", http.HandlerFunc(Cmdline))
	http.Handle("/debug/pprof/profile", http.HandlerFunc(Profile))
	http.Handle("/debug/pprof/symbol", http.HandlerFunc(Symbol))
	http.Handle("/debug/pprof/trace", http.HandlerFunc(Trace))
}
```

可以通过`runtime.GOMAXPROCS(N)`来控制Go使用的逻辑CPU数目，逻辑CPU的数目可以通过
`runtime.NumCPU()`获取，`runtime.GOMAXPROCS(N)`的源码中提到：

```go
// GOMAXPROCS sets the maximum number of CPUs that can be executing
// simultaneously and returns the previous setting.  If n < 1, it does not
// change the current setting.
// The number of logical CPUs on the local machine can be queried with NumCPU.
// This call will go away when the scheduler improves.
```

随着`goroutine scheduler`的升级，`runtime.GOMAXPROCS(N)`会被去掉

后面的章节希望能深入调研一下这里面参数的意义，能不能有比较好用的工具来自动分析
和预测一些问题.

## 4：数据完整性和一致性

本章介绍Go中的`sync`包中的`mutex`使用技巧和源码实现.

Go中`sync`包下有2种`mutex`实现：

* `sync.Mutex`

* `sync.RWMutex`

`Mutex`底层基于`sync/atomic`实现了
[Compare and Swap](https://en.wikipedia.org/wiki/Compare-and-swap).
由于该算法逻辑只需要一条汇编就可以实现，在单核CPU上运行是可以保证原子性的，但多
核CPU上运行时，需要加上`LOCK`前缀来对总线加锁，从而保证了该指令的原子性：

```go
// src/sync/atomic/asm_amd64.s#L35
TEXT ·CompareAndSwapInt32(SB),NOSPLIT,$0-17
    JMP ·CompareAndSwapUint32(SB)

TEXT ·CompareAndSwapUint32(SB),NOSPLIT,$0-17
    // 初始化参数
    MOVQ    addr+0(FP), BP
    MOVL    old+8(FP), AX
    MOVL    new+12(FP), CX
    // 锁总线
    LOCK
    // 执行Compare and Exchange
    CMPXCHGL    CX, 0(BP)
    // 处理返回值
    SETEQ   swapped+16(FP)
    RET
```

`sync.Mutex` 实现了 `sync.Locker` 接口，主要有`Lock()/Unlock()`2个method，值得注意
的是，不能重复的对已经解锁的mutex解锁，否则会`panic`.

`sync.Mutex`阻塞进程的方式其实是让进程不断的轮询.

值得注意的是`Mutex`的`zero value`是一个`unlocked mutex`:

```go
// A Mutex is a mutual exclusion lock.
// Mutexes can be created as part of other structures;
// the zero value for a Mutex is an unlocked mutex.
//
// A Mutex must not be copied after first use.
```

**TODO**

在加锁和解锁的时候，都会检查是否`enabled race`，允许抢占可能会造成不可预知的问题
看起来只有在`sync.RWMutex`和`sync.WaitGroup`里面才使用了`race.Enable()`，目前
还不清楚这个东西的作用是什么.

Go1.6中引入了更可靠的竞争检测机制，
[Introducing the Go Race Detector](https://blog.golang.org/race-detector).
只要执行`Go Command`的时候带上`-race`即可：

例4.1 `concurrency/e4_1.go` 在没有加锁的情况下存在多个`goroutine`同时读写公有
数据


```
go run -race concurrency/e4_1.go

...
Found 2 data race(s)
```

另外, `sync.RWMutex`，同样实现了`sync.Locker`接口，提供了更灵活的锁机制：

```go
// An RWMutex is a reader/writer mutual exclusion lock.
// The lock can be held by an arbitrary number of readers or a single writer.
// RWMutexes can be created as part of other structures;
// the zero value for a RWMutex is an unlocked mutex.
//
// An RWMutex must not be copied after first use.
//
// If a goroutine holds a RWMutex for reading, it must not expect this or any
// other goroutine to be able to also take the read lock until the first read
// lock is released. In particular, this prohibits recursive read locking.
// This is to ensure that the lock eventually becomes available;
// a blocked Lock call excludes new readers from acquiring the lock.
```

有意思的是，`sync.RWMutex`会检测公有数据的修改；
例4.2 `concurrency/e4_2.go`中模拟了2组`goroutine`，一组专门读数据，一组专门
写数据；其中读数据的速度非常快，写数据会比较慢（加了随机延迟）：

```go
package main

import (
	"math/rand"
	"sync"
	"time"
)

var shared = struct {
	*sync.RWMutex
	count int
}{}

var wg *sync.WaitGroup

const N = 10

func main() {
	rand.Seed(time.Now().Unix())
	shared.RWMutex = new(sync.RWMutex)
	wg = new(sync.WaitGroup)
	wg.Add(2 * N)
	defer wg.Wait()

	for i := 0; i < N; i++ {
		// write goroutines
		go func(ii int) {
			shared.Lock()
			duration := rand.Intn(5)
			// shared.Lock()
			time.Sleep(time.Duration(duration) * time.Second)
			shared.count++
			println(ii, "write --- shared.count =>", shared.count)
			// shared.Unlock()
			shared.Unlock()

			wg.Done()
		}(i)

		// read goroutines
		go func(ii int) {
			shared.RLock()
			println(ii, "read --- shared.count =>", shared.count)
			shared.RUnlock()

			wg.Done()
		}(i)

	}
}
```

读写锁的效果就是，在写锁锁定的时候，会阻塞所有的读锁，
例4.2里面读操作非常快，所以第一次写操作完成之后，
所有的读操作就一次完成了，后面的就只有读操作了

例4.3 `concurrency/e4_3.go` 去掉了公有数据的读写操作，
模拟了例4.2里面的延迟和锁；本来以为锁会检查逻辑里面的数据修改，发现并不是；
看起来写锁对读锁的阻塞是全局的，只要一个进程内的写锁就会阻塞所有的读锁；

从底层实现上来看，上面的判断应该是准确的:

```go
// src/sync/rwmutex.go#L35
// RLock locks rw for reading.
func (rw *RWMutex) RLock() {
	if race.Enabled {
		_ = rw.w.state
		race.Disable()
	}
	if atomic.AddInt32(&rw.readerCount, 1) < 0 {
		// A writer is pending, wait for it.
		runtime_Semacquire(&rw.readerSem)
	}
	if race.Enabled {
		race.Enable()
		race.Acquire(unsafe.Pointer(&rw.readerSem))
	}
}
```

操作系统只对`goroutine`实际使用的系统线程分配资源，而且`goroutine`实现了进程中
上线文切换机制，比操作系统级切换系统线程上下文高效；尽管这样，并发过高的情况下
`goroutine`之间上下文切换也会对程序性能带来比较大的影响;





## 5：锁、阻塞和Channel

[Concurrency is not parallelism](https://blog.golang.org/concurrency-is-not-parallelism)区别了`并发`和`并行`2个概念.

[Golang channels tutorial](http://guzalexander.com/2013/12/06/golang-channels-tutorial.html)
和[go语言channel的别样用法](http://studygolang.com/articles/609)
从不同的角度对`channel`的用法做了总结.

例5.1 `concurrency/e5_1.go` 介绍了在`channel`中传递`func`对象的使用.

通过这种方法可以在`goroutine`之间传递callback实现很多有趣的功能.

例5.2 `concurrency/e5_2.go` 介绍了在`channel`中传递`interface`对象.

当然也可以在`channel`中传递`struct`对象.

例5.3 `concurrency/e5_3.go` 介绍了channel作为返回值的用法.

`Channel`的零值默认是不起作用的，读写没有初始化的`channel`不会报错，
但可能出现不预期的问题.

例5.4 `concurrency/e5_4.go` 介绍了`range on channel`，`channel`在被关闭之前
是可以多次传输消息的.

这里例子很有意思，在`channel`上进行遍历的时候，像是开启了一个监听端口，
可以一直接收发送过来的消息，但这个`channel`如果用完后需要手动关闭，
因为`range`显然不可能什么时候应该终止监听.

例5.5 `concurrency/e5_5.go` 介绍了`buffered channel`

这个例子很有意思，`buffered channel`在buffer还没有满之前，是不会阻塞发送方的

```
package main

func main() {
	c := make(chan string, 1) // 这里定义了buffer的容量为1
	c <- "send message!"
	println(<-c)
}
```

上面的代码只要稍微改一下就会死锁:

```
package main

func main() {
	c := make(chan string) // 这里去掉了buffer，意味着必须要发送者接收者同时就位
	c <- "send message!"
	println(<-c)
}
```

本质上，`buffered channel`是一种 生产者－消费者 模型，所以接受者在buffer
为空时被阻塞，而生产者在buffer填满时被阻塞.

例5.6 `concurrency/e5_6.go` 介绍了`close channel`的妙用

`channel`在关闭以后，如果尝试读取，会返回`zero value of channel type`，并且不会
阻塞线程，这经常被用来实现同步，有点像`sync.WaitGroup`. 通常来说Go中实现并发和
同步的时候都应该优先考虑`channel`，它是一等公民.

`close`还可以用以从死锁中恢复，一旦`close`，所有的线程不在阻塞，可以继续执行.

例5.7 `concurrency/e5_7.go` 介绍了`select`的用法，集中处理多个`channel`的接收

为了能一直监听一系列`channel`，可以在`for`循环里面`select`

```
for {
  select {
    case <- c1:
      ...
    case <- c2:
      ...
  }
}
```


Go并发的测试和日志
----------------------

例9.1 `concurrency/e9_1.go`介绍了Go里面的异常处理

```
package main

func main() {
	defer func() {
		if err := recover(); err != nil {
			println("unexpected panic:", err)
		}
	}()

	panic("hello, recover")
}
```

Go并发编程最佳实践
------------------

例10.1 `concurrency/e10_1.go`介绍`timeout channels`,
用`time.Timer`来实现灵活的`timeout`

```
package main

import "time"

func main() {
	timer := time.NewTimer(2 * time.Second)

	defer println("timeout")

	for {
		select {
		case <-timer.C:
			return
		}
	}
}
```
