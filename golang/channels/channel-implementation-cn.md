# Go Channel 源码分析

original: <https://github.com/yangyuqian/technical-articles/blob/master/go/channel-implementation-cn.md>

传统意义上的并发编程，需要直接共享内存，这给并发模型带来了额外的复杂度.

`Goroutine` 本质上也是多线程模型，让 Go 变得特别的地方就是 `channel`，采用消息模型避免直接的内存共
享，降低了信息处理的复杂度.

注意：**本文所有的源码分析基于 Go1.6.2，版本不同实现可能会有出入**

`Channel` 是 Go 中一种特别的数据结构，它不像普通的类型一样，强调传值还是指针；对于 `channel` 而言，
只涉及传指针，因为 `make(chan)` 返回的是指针类型：

```go
// concurrency/e5_1.go
package main

func abstractListener(fxChan chan func() string) {
    fxChan <- func() string {
        return "Hello, functions in channel"
    }
}

func main() {
    fxChan := make(chan func() string)
    defer close(fxChan)

    go abstractListener(fxChan)
    select {
    case rfx := <-fxChan:
        msg := rfx()
        println("recieved message:", msg)
    }
}
```

本文从 Go Runtime 的实现角度，粗略介绍了 `channel` 的实现，希望能为其他的语言带来一些启发.

### make channel

这是初始化 channel 的过程，编译器会把 `make(chan xxx)` 最终指向：

```go
// src/runtime/chan.go#L52
func reflect_makechan(t *chantype, size int64) *hchan {
    return makechan(t, size)
}
```

最终返回一个 `*hchan` 类型，可见 `make` 出来的 `channel` 对象确实是一个指针.  `hchan` 保存了一个 
`channel` 实例必须的信息. 里面的参数后面会详细说到.

```go
// src/runtime/chan.go#L25
type hchan struct {
    qcount   uint           // buffer中数据总数
    dataqsiz uint           // buffer的容量
    buf      unsafe.Pointer // buffer的开始指针
    elemsize uint16         // channel中数据的大小
    closed   uint32         // channel是否关闭，0 => false，其他都是true
    elemtype *_type         // channel数据类型
    sendx    uint           // buffer中正在send的element的index
    recvx    uint           // buffer中正在recieve的element的index
    recvq    waitq          // 接收者等待队列
    sendq    waitq          // 发送者等待队列

    lock     mutex          // 互斥锁
}
```

代码中调用 `make(chan $type, $size)` 的时候，最终会执行 `runtime.makechan`

```go
// src/runtime/chan.go#L56
func makechan(t *chantype, size int64) *hchan {
    // 初始化，做一些基本校验

    var c *hchan
    // 如果是无指针类型或者指定 size=0；可见默认情况下都会走这个逻辑
    if elem.kind&kindNoPointers != 0 || size == 0 {
        // 从内存分配器请求一段内存
        c = (*hchan)(mallocgc(hchanSize+uintptr(size)*elem.size, nil, true))
        // 如果是无指针类型，并且指定了 buffer size
        if size > 0 && elem.size != 0 {
            c.buf = add(unsafe.Pointer(c), hchanSize)
        } else {
            // race detector uses this location for synchronization
            // Also prevents us from pointing beyond the allocation (see issue 9401).
            c.buf = unsafe.Pointer(c)
        }
    } else {
        // 有指针类型，并且 size > 0 时，直接初始化一段内存
        c = new(hchan)
        c.buf = newarray(elem, int(size))
    }
    // 初始化 channel 结构里面的参数
    c.elemsize = uint16(elem.size)
    c.elemtype = elem
    c.dataqsiz = uint(size)

    // ...
    return c
}
```

上面的代码提到了“无指针类型”，一个比较简单的例子就是反射中的值类型，这种值是没办法直接获取地址的, 也
有些地方叫做“不可寻址”：

```go
i := 1
// 从道理上上，这个 value 实际上就是一个抽象概念，
// 好比从一个活人身上把肉体抽出来 => value
// 脱离了具体的对象，所以就无法谈地址，但这样的值却实实在在存在
v := reflect.ValueOf(i)
// 然后灵魂也可以抽出来 => type
t := reflect.TypeOf(i)
```

### send to channel

初始化 `channel` 之后就可以向 `channel` 中写数据，准确的说，应该是发送数据.

入口函数：

```go
// src/runtime/chan.go#L106
func chansend1(t *chantype, c *hchan, elem unsafe.Pointer) {
    chansend(t, c, elem, true, getcallerpc(unsafe.Pointer(&t)))
}
```

核心逻辑都在 `chansend` 中实现，注意这里的 `block:bool` 参数始终为 `true`:

```go
// src/runtime/chan.go#L122
func chansend(t *chantype, c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
    // 初始化，异常校验和处理

    // 处理 channel 的临界状态，既没有被 close，又不能正常接收数据
    if !block && c.closed == 0 && ((c.dataqsiz == 0 && c.recvq.first == nil) ||
        (c.dataqsiz > 0 && c.qcount == c.dataqsiz)) {
        return false
    }

    var t0 int64
    if blockprofilerate > 0 {
        t0 = cputicks()
    }

    // 对同一个 channel 的操作是串行的
    lock(&c.lock)

    // 向一个已经被 close 的 channel 写数据，会 panic 
    if c.closed != 0 {
        unlock(&c.lock)
        panic(plainError("send on closed channel"))
    }

    // 这是一个不阻塞的处理，如果已经有接收者，
    // 就向第一个接收者发送当前 enqueue 的消息.
    // return true 就是说发送成功了，写入 buffer 也算是成功
    if sg := c.recvq.dequeue(); sg != nil {
        // 发送的实现本质上就是在不同线程的栈上 copy 数据
        send(c, sg, ep, func() { unlock(&c.lock) })
        return true
    }

    if c.qcount < c.dataqsiz {
        // 把发送的数据写入 buffer，更新 buffer 状态，返回成功
    }

    // 如果buffer满了，或者non-buffer（等同于buffer满了），
    // 阻塞sender并更新一些栈的状态，
    // 唤醒线程的时候还会重新检查channel是否打开状态，否则panic
}
```

再看 `send` 的实现:

```go
// src/runtime/chan.go#L122
func send(c *hchan, sg *sudog, ep unsafe.Pointer, unlockf func()) {
    if sg.elem != nil {
        // 直接把要发送的数据 copy 到 reciever 的栈空间
        sendDirect(c.elemtype, sg, ep)
        sg.elem = nil
    }

  // 等待 reciever 就绪，如果 reciever 还没准备好就阻塞 sender 一段时间
}
```

`sendDirect` 的实现：

```go
// src/runtime/chan.go#L284
func sendDirect(t *_type, sg *sudog, src unsafe.Pointer) {
    // 直接拷贝数据
    dst := sg.elem
    memmove(dst, src, t.size)
    typeBitsBulkBarrier(t, uintptr(dst), t.size)
}
```

`channel send` 主要就是做了2件事情：

* 如果没有可用的 `reciever`，数据入队（如果有 buffer），否则线程阻塞
* 如果有可用的 `reciever`，就把数据从 `sender` 的栈空间拷贝到 `reciever` 的栈空间

### recieve from channel

入口函数：

```go
// src/runtime/chan.go#L377
func chanrecv1(t *chantype, c *hchan, elem unsafe.Pointer) {
    chanrecv(t, c, elem, true)
}
```

`chanrecv` 的实现：

```go
// src/runtime/chan.go#L393
func chanrecv(t *chantype, c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {
    // 初始化，处理基本校验

    // 处理 channel 临界状态， reciever 在接收之前要保证 channel 是就绪的
    if !block && (c.dataqsiz == 0 && c.sendq.first == nil ||
        c.dataqsiz > 0 && atomic.Loaduint(&c.qcount) == 0) &&
        atomic.Load(&c.closed) == 0 {
        return
    }

    // ...

    // 如果 recieve 数据的时候，发现 channel 关闭了，
    // 直接返回 channel type 的 zero value
    if c.closed != 0 && c.qcount == 0 {
        // ...
        if ep != nil {
            memclr(ep, uintptr(c.elemsize))
        }
        return true, false
    }

    // 从 sender 阻塞队首获取 sender，接收数据
    // 如果 buffer 为空，直接获取 sender 的数据，否则把 sender 的数据加到 buffer
    // 队尾，然后从 buffer 队首获取数据
    if sg := c.sendq.dequeue(); sg != nil {
        recv(c, sg, ep, func() { unlock(&c.lock) })
        return true, true
    }

    // 如果当前没有 pending 的 sender ，且 buffer 里有数据，直接从 buffer 里面拿数据
    if c.qcount > 0 {
        // ...
    }

    // buffer 里面没有数据，也没有阻塞的 sender，就阻塞 reciver
}
```

### close channel

入口函数：

```go
// src/runtime/chan.go#L303
func closechan(c *hchan) {
    // 关闭 channel 之前必须已经初始化

    // 检查 channel 状态，不能重复关闭 channel

    // ...

    // 将closed置为1，标志channel已经关闭
    c.closed = 1

    // ...

    // 先释放所有的 reciever/reader

    // 然后释放所有的 sender/writer，在 sender 端会 panic

    // ...
}
```
