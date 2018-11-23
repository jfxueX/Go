# Concurrency in Go

by Katherine Cox-Buday

# Index

-----------------------------
* [Symbols](#symbols)
* [A](#a)
* [B](#b)
* [C](#c)
* [D](#d)
* [E](#e)
* [F](#f)
* [G](#g)
* [H](#h)
* [I](#i)
* [L](#l)
* [M](#m)
* [N](#n)
* [O](#o)
* [P](#p)
* [Q](#q)
* [R](#r)
* [S](#s)
* [T](#t)
* [U](#u)
* [W](#w)
-----------------------------


### Symbols

  - <span data-type="index-term">:= operator</span>,
    [Channels](ch03.html#idm140183153238064)
  - <span data-type="index-term">\<- operator</span>,
    [Channels](ch03.html#idm140183153198768)

</div>

<div data-type="indexdiv">

### A

  - <span data-type="index-term">acknowledgments</span>,
    [Acknowledgments](preface01.html#idm140183166978064)
  - <span data-type="index-term">additional resources</span>, [Online
    Resources](preface01.html#idm140183166961152)
  - <span data-type="index-term">Amdahl, Gene</span>, [Moore’s Law, Web
    Scale, and the Mess We’re In](ch01.html#idm140183167023056)
  - <span data-type="index-term">Amdahl’s law</span>, [Moore’s Law, Web
    Scale, and the Mess We’re In](ch01.html#idm140183167021728)
  - <span data-type="index-term">anonymous functions</span>,
    [Goroutines](ch03.html#idm140183157397488)
  - <span data-type="index-term">atomicity</span>,
    [Atomicity](ch01.html#atom01)-[Atomicity](ch01.html#idm140183163860560)

</div>

<div data-type="indexdiv">

### B

  - <span data-type="index-term">batch processing</span>,
    [Pipelines](ch04.html#idm140183147916944)
  - <span data-type="index-term">bridge-channels</span>, [The
    bridge-channel](ch04.html#idm140183144678816)
  - <span data-type="index-term">Broadcast method</span>,
    [Cond](ch03.html#idm140183155260352)
  - <span data-type="index-term">bugs</span>, [Error
    Propagation](ch05.html#idm140183143008192)
      - (<span data-gentext="see">see also</span> error handling)

</div>

<div data-type="indexdiv">

### C

  - <span data-type="index-term">cancellations</span>
    (<span data-gentext="see">see</span> timeouts and cancellations)
  - <span data-type="index-term">channels</span>
      - <span data-type="index-term">best use of</span>,
        [Channels](ch03.html#idm140183153248512)
      - <span data-type="index-term">bidirectional</span>,
        [Channels](ch03.html#idm140183153149456)
      - <span data-type="index-term">blocking channels</span>,
        [Channels](ch03.html#idm140183152831360),
        [Queuing](ch04.html#idm140183144119056)
      - <span data-type="index-term">bridge-channels</span>, [The
        bridge-channel](ch04.html#idm140183144678144)
      - <span data-type="index-term">buffered</span>,
        [Channels](ch03.html#Cbuff03)-[Channels](ch03.html#idm140183151915600)
      - <span data-type="index-term">closing</span>,
        [Channels](ch03.html#idm140183152639552)
      - <span data-type="index-term">creating</span>,
        [Channels](ch03.html#idm140183153239376)
      - <span data-type="index-term">data types constraints</span>,
        [Channels](ch03.html#idm140183153129984)
      - <span data-type="index-term">history of</span>, [The Difference
        Between Concurrency and
        Parallelism](ch02.html#idm140183157940720)
      - <span data-type="index-term">multiple goroutines</span>,
        [Channels](ch03.html#idm140183152428384), [Fan-Out,
        Fan-In](ch04.html#idm140183146203984)
      - <span data-type="index-term">mutexes and</span>, [How This Helps
        You](ch02.html#idm140183157635280)
      - <span data-type="index-term">nil</span>,
        [Channels](ch03.html#idm140183151914480)
      - <span data-type="index-term">operations reference chart</span>,
        [Channels](ch03.html#idm140183151904480)
      - <span data-type="index-term">or-channel</span>, [The
        or-channel](ch04.html#Cor04)-[The
        or-channel](ch04.html#idm140183149035904)
      - <span data-type="index-term">or-done-channels</span>, [The
        or-done-channel](ch04.html#idm140183145447104)
      - <span data-type="index-term">ownership assignment</span>,
        [Channels](ch03.html#idm140183151708512)
      - <span data-type="index-term">ranging over</span>,
        [Channels](ch03.html#idm140183152553968)
      - <span data-type="index-term">reading from</span>,
        [Channels](ch03.html#idm140183152797152)
      - <span data-type="index-term">reading from closed</span>,
        [Channels](ch03.html#idm140183152600560)
      - <span data-type="index-term">tee-channels</span>, [The
        tee-channel](ch04.html#idm140183145024208)
      - <span data-type="index-term">unbuffered</span>,
        [Channels](ch03.html#idm140183152220368)
      - <span data-type="index-term">unidirectional</span>,
        [Channels](ch03.html#idm140183153199744)
      - <span data-type="index-term">writing to</span>,
        [Channels](ch03.html#idm140183152730720)
  - <span data-type="index-term">checkStatus</span>, [Error
    Handling](ch04.html#checkstat04)-[Error
    Handling](ch04.html#idm140183148387536)
  - <span data-type="index-term">chunking</span>,
    [Queuing](ch04.html#idm140183143918896)
  - <span data-type="index-term">circular wait</span>,
    [Deadlock](ch01.html#idm140183163374880)
  - <span data-type="index-term">cloud computing</span>, [Moore’s Law,
    Web Scale, and the Mess We’re In](ch01.html#idm140183162930352)
  - <span data-type="index-term">code examples, obtaining and
    using</span>, [Using Code
    Examples](preface01.html#idm140183167005648)
  - <span data-type="index-term">Coffman Conditions</span>,
    [Deadlock](ch01.html#idm140183163398400)
  - <span data-type="index-term">Coffman, Edgar</span>,
    [Deadlock](ch01.html#idm140183163397696)
  - <span data-type="index-term">comments and questions</span>, [How to
    Contact Us](preface01.html#idm140183166993696)
  - <span data-type="index-term">community-oriented resources</span>,
    [Online Resources](preface01.html#idm140183166961840)
  - <span data-type="index-term">concurrency</span>
      - <span data-type="index-term">common issues</span>
          - <span data-type="index-term">atomicity</span>,
            [Atomicity](ch01.html#Cciatom01)-[Atomicity](ch01.html#idm140183163861536)
          - <span data-type="index-term">deadlocks</span>,
            [Deadlock](ch01.html#Ccidead01)-[Deadlock](ch01.html#idm140183163361120)
          - <span data-type="index-term">determining concurrency
            safety</span>, [Determining Concurrency
            Safety](ch01.html#Ccisafety01)-[Determining Concurrency
            Safety](ch01.html#idm140183157834224),
            [Confinement](ch04.html#idm140183150924960)
          - <span data-type="index-term">livelocks</span>,
            [Livelock](ch01.html#Ccilive01)-[Livelock](ch01.html#idm140183163277856)
          - <span data-type="index-term">memory access
            synchronization</span>, [Memory Access
            Synchronization](ch01.html#Ccimemory01)-[Memory Access
            Synchronization](ch01.html#idm140183163634336),
            [Starvation](ch01.html#idm140183158026544), [What Is
            CSP?](ch02.html#idm140183157650064), [The sync
            Package](ch03.html#idm140183156316656)
          - <span data-type="index-term">race conditions</span>, [Race
            Conditions](ch01.html#Ccirace01)-[Race
            Conditions](ch01.html#idm140183163954320), [Race
            Detection](app01.html#idm140183133315424)
          - <span data-type="index-term">starvation</span>,
            [Starvation](ch01.html#Ccistarv01)-[Starvation](ch01.html#idm140183158021008)
      - <span data-type="index-term">coroutines and</span>,
        [Goroutines](ch03.html#idm140183157448944)
      - <span data-type="index-term">definition of term</span>, [An
        Introduction to Concurrency](ch01.html#idm140183167035520)
      - <span data-type="index-term">fork-join model</span>,
        [Goroutines](ch03.html#Cgomodel03)-[Goroutines](ch03.html#idm140183156320992)
      - <span data-type="index-term">Go’s approach to</span>, [How This
        Helps You](ch02.html#Cgo02)-[How This Helps
        You](ch02.html#idm140183157605904)
      - <span data-type="index-term">Go’s philosophy on</span>, [Go’s
        Philosophy on Concurrency](ch02.html#Cphilosophy02)-[Go’s
        Philosophy on Concurrency](ch02.html#idm140183157523344)
      - <span data-type="index-term">history of</span>, [Moore’s Law,
        Web Scale, and the Mess We’re In](ch01.html#Chistory01)
      - <span data-type="index-term">vs. parallelism</span>, [The
        Difference Between Concurrency and
        Parallelism](ch02.html#Cparallel02)-[The Difference Between
        Concurrency and Parallelism](ch02.html#idm140183157696176), [How
        This Helps You](ch02.html#idm140183157619328)
  - <span data-type="index-term">concurrency patterns</span>
      - <span data-type="index-term">bridge-channels</span>, [The
        bridge-channel](ch04.html#idm140183144679792)
      - <span data-type="index-term">confinement</span>,
        [Confinement](ch04.html#CPconfine04)-[Confinement](ch04.html#idm140183150462976)
      - <span data-type="index-term">context package</span>, [The
        context Package](ch04.html#CPcontext04)-[The context
        Package](ch04.html#idm140183140131856)
      - <span data-type="index-term">empty interfaces and</span>,
        [Concurrency Patterns in Go](ch04.html#idm140183150972208)
      - <span data-type="index-term">error handling</span>, [Error
        Handling](ch04.html#CPerror04)-[Error
        Handling](ch04.html#idm140183148388448)
      - <span data-type="index-term">fan-out, fan-in</span>, [Fan-Out,
        Fan-In](ch04.html#CPfan04)-[Fan-Out,
        Fan-In](ch04.html#idm140183145550688)
      - <span data-type="index-term">for-select loops</span>, [The
        for-select Loop](ch04.html#idm140183150460480)
      - <span data-type="index-term">or-channel</span>, [The
        or-channel](ch04.html#CPor04)-[The
        or-channel](ch04.html#idm140183149050160)
      - <span data-type="index-term">or-done-channels</span>, [The
        or-done-channel](ch04.html#idm140183145448976)
      - <span data-type="index-term">pipelines</span>,
        [Pipelines](ch04.html#CPpipe04)-[Some Handy
        Generators](ch04.html#idm140183146212272)
      - <span data-type="index-term">preventing goroutine leaks</span>,
        [Preventing Goroutine Leaks](ch04.html#CPleaks04)-[Preventing
        Goroutine Leaks](ch04.html#idm140183149539408)
      - <span data-type="index-term">queuing</span>,
        [Queuing](ch04.html#CPque04)-[Queuing](ch04.html#idm140183143870112)
      - <span data-type="index-term">tee-channels</span>, [The
        tee-channel](ch04.html#idm140183145025856)
  - <span data-type="index-term">concurrency primitives</span>
      - <span data-type="index-term">channels</span>,
        [Channels](ch03.html#CPchannel03)-[Channels](ch03.html#idm140183151486688)
      - <span data-type="index-term">Cond type</span>,
        [Cond](ch03.html#CPcond03)-[Cond](ch03.html#idm140183154951664)
      - <span data-type="index-term">GOMAXPROCS function</span>, [The
        GOMAXPROCS Lever](ch03.html#idm140183150987568)
      - <span data-type="index-term">goroutines</span>,
        [Goroutines](ch03.html#CPgo04)-[Goroutines](ch03.html#idm140183156320048)
      - <span data-type="index-term">select statement</span>, [How This
        Helps You](ch02.html#idm140183157609776)
      - <span data-type="index-term">select statements</span>, [The
        select Statement](ch03.html#CPselect04)-[The select
        Statement](ch03.html#idm140183150990272)
      - <span data-type="index-term">sync package</span>, [The sync
        Package](ch03.html#CPsync04)-[Pool](ch03.html#idm140183153252128)
          - <span data-type="index-term">Cond type</span>,
            [Cond](ch03.html#idm140183155717616)
          - <span data-type="index-term">Mutex and RWMutex</span>,
            [Mutex and RWMutex](ch03.html#idm140183156032512)
          - <span data-type="index-term">once variable</span>,
            [Once](ch03.html#idm140183154947424)
          - <span data-type="index-term">Pool</span>,
            [Pool](ch03.html#idm140183154158592)
          - <span data-type="index-term">WaitGroup</span>,
            [WaitGroup](ch03.html#idm140183156307424)
  - <span data-type="index-term">Cond type</span>,
    [Cond](ch03.html#cond03)-[Cond](ch03.html#idm140183154952608)
  - <span data-type="index-term">confinement</span>
      - <span data-type="index-term">ad-hoc</span>,
        [Confinement](ch04.html#idm140183150913744)
      - <span data-type="index-term">benefits and drawbacks of</span>,
        [Confinement](ch04.html#idm140183150916512)
      - <span data-type="index-term">immutable data and</span>,
        [Confinement](ch04.html#idm140183150926576)
      - <span data-type="index-term">lexical</span>,
        [Goroutines](ch03.html#idm140183157214736),
        [Once](ch03.html#idm140183154312416),
        [Confinement](ch04.html#Clexical04)-[Confinement](ch04.html#idm140183150462000)
  - <span data-type="index-term">contact information</span>, [How to
    Contact Us](preface01.html#idm140183166995072)
  - <span data-type="index-term">context</span>,
    [Atomicity](ch01.html#context01)-[Atomicity](ch01.html#idm140183163859616),
    [The Difference Between Concurrency and
    Parallelism](ch02.html#idm140183157959344)
  - <span data-type="index-term">context package</span>
      - <span data-type="index-term">benefits of</span>, [The context
        Package](ch04.html#idm140183143867440)
      - <span data-type="index-term">cancelling call-graph branches
        with</span>, [The context Package](ch04.html#idm140183143452352)
      - <span data-type="index-term">Context type</span>, [The context
        Package](ch04.html#idm140183143857440)
      - <span data-type="index-term">Deadline method</span>, [The
        context Package](ch04.html#CPdead04)-[The context
        Package](ch04.html#idm140183138430800)
      - <span data-type="index-term">vs. done channel pattern</span>,
        [The context Package](ch04.html#CPdone04)-[The context
        Package](ch04.html#idm140183141577184)
      - <span data-type="index-term">Done method</span>, [The context
        Package](ch04.html#idm140183143642192)
      - <span data-type="index-term">example</span>, [The context
        Package](ch04.html#idm140183143551312)
      - <span data-type="index-term">functions in</span>, [The context
        Package](ch04.html#idm140183143441072)
      - <span data-type="index-term">guidelines for use</span>, [The
        context Package](ch04.html#idm140183141948112)
      - <span data-type="index-term">purposes served by</span>, [The
        context Package](ch04.html#idm140183143456336)
      - <span data-type="index-term">transporting request-scoped data
        with</span>, [The context Package](ch04.html#CPtrans04)-[The
        context Package](ch04.html#idm140183141949344)
  - <span data-type="index-term">context switching</span>,
    [Goroutines](ch03.html#idm140183156618208)
  - <span data-type="index-term">coroutines</span>,
    [Goroutines](ch03.html#idm140183157452384)
  - <span data-type="index-term">critical sections</span>, [Memory
    Access Synchronization](ch01.html#criticalsec01)-[Memory Access
    Synchronization](ch01.html#idm140183163636224)
  - <span data-type="index-term">CSP (Communicating Sequential
    Processes)</span>
      - <span data-type="index-term">concept of</span>, [What Is
        CSP?](ch02.html#CSPconept02)-[What Is
        CSP?](ch02.html#idm140183157645904)
      - <span data-type="index-term">concurrency vs. parallelism</span>,
        [The Difference Between Concurrency and
        Parallelism](ch02.html#CSPconcurrent02)-[The Difference Between
        Concurrency and Parallelism](ch02.html#idm140183157935200)
      - <span data-type="index-term">Go’s approach to
        concurrency</span>, [How This Helps
        You](ch02.html#CSPgoapproach02)-[How This Helps
        You](ch02.html#idm140183157604960)
      - <span data-type="index-term">Go’s philosophy on
        concurrency</span>, [Go’s Philosophy on
        Concurrency](ch02.html#CSPphilosophy02)-[Go’s Philosophy on
        Concurrency](ch02.html#idm140183157522400)

</div>

<div data-type="indexdiv">

### D

  - <span data-type="index-term">data race</span>, [Race
    Conditions](ch01.html#datarace01)-[Race
    Conditions](ch01.html#idm140183163956208)
  - <span data-type="index-term">databases, goroutine cancellations
    and</span>, [Timeouts and
    Cancellation](ch05.html#idm140183139490336)
  - <span data-type="index-term">DDoS (distributed denial of service)
    attacks</span>, [Rate Limiting](ch05.html#idm140183137887920)
  - <span data-type="index-term">deadlocks</span>,
    [Deadlock](ch01.html#dead01)-[Deadlock](ch01.html#idm140183163362096),
    [Timeouts and Cancellation](ch05.html#idm140183142160784)
  - <span data-type="index-term">death-spirals</span>,
    [Queuing](ch04.html#idm140183143911520)
  - <span data-type="index-term">debugging</span>
      - <span data-type="index-term">goroutine errors</span>, [Anatomy
        of a Goroutine Error](app01.html#idm140183133362400)
      - <span data-type="index-term">healing unhealthy
        goroutines</span>, [Healing Unhealthy
        Goroutines](ch05.html#DBheal05)-[Healing Unhealthy
        Goroutines](ch05.html#idm140183134046384)
      - <span data-type="index-term">pprof profiler</span>,
        [pprof](app01.html#idm140183133201104)
      - <span data-type="index-term">race detection</span>, [Race
        Detection](app01.html#idm140183133314208)
  - <span data-type="index-term">decision-tree</span>, [Go’s Philosophy
    on Concurrency](ch02.html#idm140183157582400)
  - <span data-type="index-term">Dijkstra, Edgar</span>, [What Is
    CSP?](ch02.html#idm140183157657392)
  - <span data-type="index-term">double-ended queues (deques)</span>,
    [Work Stealing](ch06.html#idm140183134109520)
  - <span data-type="index-term">duplicate messages</span>, [Timeouts
    and Cancellation](ch05.html#duplicate05)-[Timeouts and
    Cancellation](ch05.html#idm140183142372112)

</div>

<div data-type="indexdiv">

### E

  - <span data-type="index-term">embarrassingly parallel</span>,
    [Moore’s Law, Web Scale, and the Mess We’re
    In](ch01.html#idm140183167018832)
  - <span data-type="index-term">empty interfaces (interface{})</span>
      - <span data-type="index-term">benefits of</span>, [Concurrency
        Patterns in Go](ch04.html#idm140183150973152)
      - <span data-type="index-term">chan interface{} variable</span>,
        [Channels](ch03.html#idm140183153206400)
      - <span data-type="index-term">pipeline interfaces</span>, [Some
        Handy Generators](ch04.html#idm140183146568848)
  - <span data-type="index-term">errata</span>, [How to Contact
    Us](preface01.html#idm140183166989120)
  - <span data-type="index-term">error handling</span>
      - <span data-type="index-term">complete example</span>, [Error
        Propagation](ch05.html#EHcomplerte05)-[Error
        Propagation](ch05.html#idm140183139602560)
      - <span data-type="index-term">components of</span>, [Error
        Propagation](ch05.html#idm140183143021088)
      - <span data-type="index-term">displaying errors to users</span>,
        [Error Propagation](ch05.html#idm140183141890880)
      - <span data-type="index-term">error anatomy</span>, [Anatomy of a
        Goroutine Error](app01.html#idm140183133363248)
      - <span data-type="index-term">error categories</span>, [Error
        Propagation](ch05.html#idm140183143009168)
      - <span data-type="index-term">error correctness</span>, [Error
        Propagation](ch05.html#idm140183141892800)
      - <span data-type="index-term">error packages</span>, [Error
        Propagation](ch05.html#idm140183142185008)
      - <span data-type="index-term">importance of</span>, [Error
        Propagation](ch05.html#idm140183140123200)
      - <span data-type="index-term">multiple module example</span>,
        [Error Propagation](ch05.html#idm140183142771184)
      - <span data-type="index-term">responsibility for</span>, [Error
        Handling](ch04.html#error04)-[Error
        Handling](ch04.html#idm140183148386592)

</div>

<div data-type="indexdiv">

### F

  - <span data-type="index-term">fair scheduling strategy</span>, [Work
    Stealing](ch06.html#idm140183134027680)
  - <span data-type="index-term">fan-out, fan-in technique</span>,
    [Fan-Out, Fan-In](ch04.html#fan04)-[Fan-Out,
    Fan-In](ch04.html#idm140183145549744)
  - <span data-type="index-term">files, goroutine cancellations
    and</span>, [Timeouts and
    Cancellation](ch05.html#idm140183139489696)
  - <span data-type="index-term">for-select loops</span>, [The
    for-select Loop](ch04.html#idm140183150459504)
  - <span data-type="index-term">fork-join model</span>
      - <span data-type="index-term">benefits of</span>,
        [Goroutines](ch03.html#idm140183156884560)
      - <span data-type="index-term">child branches</span>,
        [Goroutines](ch03.html#idm140183157441472)
      - <span data-type="index-term">closures and</span>,
        [Goroutines](ch03.html#idm140183157215712)
      - <span data-type="index-term">context switching and</span>,
        [Goroutines](ch03.html#idm140183156617600)
      - <span data-type="index-term">graphical representation of</span>,
        [Goroutines](ch03.html#idm140183157434064)
      - <span data-type="index-term">interdependency of tasks in</span>,
        [Work Stealing](ch06.html#idm140183134022640)
      - <span data-type="index-term">join points</span>,
        [Goroutines](ch03.html#idm140183157435552)
      - <span data-type="index-term">memory management in</span>,
        [Goroutines](ch03.html#idm140183157101744)
      - <span data-type="index-term">parent cancellations</span>,
        [Timeouts and Cancellation](ch05.html#idm140183142149248)
      - <span data-type="index-term">synchronization in</span>,
        [Goroutines](ch03.html#idm140183156887632)
      - <span data-type="index-term">tasks vs. continuations</span>,
        [Stealing Tasks or
        Continuations?](ch06.html#FJMtasks06)-[Stealing Tasks or
        Continuations?](ch06.html#idm140183133393648)

</div>

<div data-type="indexdiv">

### G

  - <span data-type="index-term">garbage collection</span>, [Simplicity
    in the Face of Complexity](ch01.html#idm140183157817296)
  - <span data-type="index-term">generators</span>, [Best Practices for
    Constructing Pipelines](ch04.html#generator04)-[Some Handy
    Generators](ch04.html#idm140183146213248)
  - <span data-type="index-term">Get method</span>,
    [Pool](ch03.html#idm140183154140016)
  - <span data-type="index-term">Go</span>
      - <span data-type="index-term">benefits of</span>, [Determining
        Concurrency Safety](ch01.html#Gbenefits01)-[Simplicity in the
        Face of Complexity](ch01.html#idm140183157803760)
      - <span data-type="index-term">channels</span>, [What Is
        CSP?](ch02.html#Gchan02)-[How This Helps
        You](ch02.html#idm140183157633856),
        [Channels](ch03.html#Gchannel03)-[Channels](ch03.html#idm140183151487536)
      - <span data-type="index-term">concurrency patterns in</span>,
        [Concurrency Patterns in
        Go](ch04.html#Gpattern04)-[Summary](ch04.html#idm140183143864160)
      - <span data-type="index-term">concurrency primitives</span>,
        [Goroutines](ch03.html#Gobuildrout03)-[Conclusion](ch03.html#idm140183150832304)
      - <span data-type="index-term">memory management in</span>,
        [Goroutines](ch03.html#idm140183157099296)
      - <span data-type="index-term">online resources</span>, [Online
        Resources](preface01.html#idm140183166963488)
      - <span data-type="index-term">origins of</span>, [Goroutines and
        the Go Runtime](ch06.html#idm140183134035456)
      - <span data-type="index-term">vs. other languages</span>, [How
        This Helps You](ch02.html#Goother02)-[How This Helps
        You](ch02.html#idm140183157606880)
      - <span data-type="index-term">philosophy on concurrency</span>,
        [Go’s Philosophy on Concurrency](ch02.html#Gphilosophy02)-[Go’s
        Philosophy on Concurrency](ch02.html#idm140183157524320)
          - <span data-type="index-term">decision-tree</span>, [Go’s
            Philosophy on Concurrency](ch02.html#idm140183157581696)
      - <span data-type="index-term">prerequisites to learning</span>,
        [Who Should Read This Book](preface01.html#idm140183166870208)
      - <span data-type="index-term">select statement</span>, [How This
        Helps You](ch02.html#idm140183157610720)
      - <span data-type="index-term">Wiki</span>, [Go’s Philosophy on
        Concurrency](ch02.html#idm140183157590608)
  - <span data-type="index-term">go keyword</span>, [Presenting All of
    This to the Developer](ch06.html#idm140183134032192)
  - <span data-type="index-term">GOMAXPROCS function</span>, [The
    GOMAXPROCS Lever](ch03.html#idm140183150988272)
  - <span data-type="index-term">goroutines</span>
      - <span data-type="index-term">benefits of</span>,
        [Goroutines](ch03.html#idm140183156924064), [Presenting All of
        This to the Developer](ch06.html#idm140183134031488)
      - <span data-type="index-term">creating</span>,
        [Goroutines](ch03.html#idm140183157510880)
      - <span data-type="index-term">error handling</span>, [Anatomy of
        a Goroutine Error](app01.html#idm140183133364096)
      - <span data-type="index-term">healing unhealthy</span>, [Healing
        Unhealthy Goroutines](ch05.html#GRheal05)-[Healing Unhealthy
        Goroutines](ch05.html#idm140183134048144)
      - <span data-type="index-term">main goroutine</span>,
        [Goroutines](ch03.html#idm140183157514336)
      - <span data-type="index-term">multiple</span>,
        [Channels](ch03.html#idm140183152427408), [Fan-Out,
        Fan-In](ch04.html#idm140183146203008)
      - <span data-type="index-term">operation of</span>,
        [Goroutines](ch03.html#GRoperation03)-[Goroutines](ch03.html#idm140183156321936)
      - <span data-type="index-term">preventing leaks</span>,
        [Preventing Goroutine Leaks](ch04.html#GRleaks04)-[Preventing
        Goroutine Leaks](ch04.html#idm140183149540352)
      - <span data-type="index-term">restarting</span>, [Healing
        Unhealthy Goroutines](ch05.html#GRrestart05)-[Healing Unhealthy
        Goroutines](ch05.html#idm140183134044496)
      - <span data-type="index-term">scheduling</span>, [Goroutines and
        the Go Runtime](ch06.html#GRschedule06)-[Presenting All of This
        to the Developer](ch06.html#idm140183133371504)
      - <span data-type="index-term">termination of</span>, [Preventing
        Goroutine Leaks](ch04.html#idm140183150181152)
  - <span data-type="index-term">guarded commands</span>, [What Is
    CSP?](ch02.html#idm140183157658320)

</div>

<div data-type="indexdiv">

### H

  - <span data-type="index-term">healing unhealthy goroutines</span>,
    [Healing Unhealthy Goroutines](ch05.html#heal05)-[Healing Unhealthy
    Goroutines](ch05.html#idm140183134045440)
  - <span data-type="index-term">heartbeats</span>
      - <span data-type="index-term">before each unit of work</span>,
        [Heartbeats](ch05.html#HBbefore05)-[Heartbeats](ch05.html#idm140183140800960)
      - <span data-type="index-term">demonstration of</span>,
        [Heartbeats](ch05.html#Hdemo05)-[Heartbeats](ch05.html#idm140183140848160)
      - <span data-type="index-term">deterministic tests using</span>,
        [Heartbeats](ch05.html#idm140183140919136)
      - <span data-type="index-term">interval-based</span>,
        [Heartbeats](ch05.html#idm140183140799840),
        [Heartbeats](ch05.html#HBinterval05)-[Heartbeats](ch05.html#idm140183137571952)
      - <span data-type="index-term">roles of</span>,
        [Heartbeats](ch05.html#idm140183142366448),
        [Heartbeats](ch05.html#idm140183140873792),
        [Heartbeats](ch05.html#idm140183140798864)
      - <span data-type="index-term">types of</span>,
        [Heartbeats](ch05.html#idm140183142364224)
  - <span data-type="index-term">higher order functions</span>,
    [Pipelines](ch04.html#idm140183148061760)
  - <span data-type="index-term">Hoare, Tony</span>, [The Difference
    Between Concurrency and Parallelism](ch02.html#hoare02)-[What Is
    CSP?](ch02.html#idm140183157646880)

</div>

<div data-type="indexdiv">

### I

  - <span data-type="index-term">immutable data</span>,
    [Confinement](ch04.html#idm140183150925632)
  - <span data-type="index-term">indivisible</span>,
    [Atomicity](ch01.html#idm140183163942016)

</div>

<div data-type="indexdiv">

### L

  - <span data-type="index-term">leaks, preventing</span>, [Preventing
    Goroutine Leaks](ch04.html#leak04)-[Preventing Goroutine
    Leaks](ch04.html#idm140183149541360)
  - <span data-type="index-term">Little’s law</span>,
    [Queuing](ch04.html#idm140183143899728)
  - <span data-type="index-term">livelocks</span>,
    [Livelock](ch01.html#live01)-[Livelock](ch01.html#idm140183163278832)

</div>

<div data-type="indexdiv">

### M

  - <span data-type="index-term">M:N scheduler</span>,
    [Goroutines](ch03.html#idm140183157446640)
  - <span data-type="index-term">make function</span>,
    [Channels](ch03.html#idm140183153202528)
  - <span data-type="index-term">memory access synchronization</span>,
    [Memory Access Synchronization](ch01.html#memory01)-[Memory Access
    Synchronization](ch01.html#idm140183163635280),
    [Starvation](ch01.html#idm140183158028128), [What Is
    CSP?](ch02.html#idm140183157651648), [The sync
    Package](ch03.html#idm140183156317296)
  - <span data-type="index-term">memory management</span>, [Simplicity
    in the Face of Complexity](ch01.html#idm140183157815328),
    [Goroutines](ch03.html#idm140183157098544)
  - <span data-type="index-term">monads</span>,
    [Pipelines](ch04.html#idm140183148061056)
  - <span data-type="index-term">Moore, Gordon</span>, [Moore’s Law, Web
    Scale, and the Mess We’re In](ch01.html#idm140183167027200)
  - <span data-type="index-term">Moore’s law</span>, [Moore’s Law, Web
    Scale, and the Mess We’re In](ch01.html#idm140183167024432)
  - <span data-type="index-term">multiplexing</span>, [Preventing
    Goroutine Leaks](ch04.html#idm140183150183072), [Fan-Out,
    Fan-In](ch04.html#idm140183145751136)
  - <span data-type="index-term">Mutex and RWMutex</span>, [Mutex and
    RWMutex](ch03.html#mutex03)-[Mutex and
    RWMutex](ch03.html#idm140183154681952)
  - <span data-type="index-term">mutexes</span>, [How This Helps
    You](ch02.html#idm140183157635952)
  - <span data-type="index-term">mutual exclusion</span>,
    [Deadlock](ch01.html#idm140183163393376)

</div>

<div data-type="indexdiv">

### N

  - <span data-type="index-term">negative feedback loops</span>,
    [Queuing](ch04.html#idm140183143912192)
  - <span data-type="index-term">no preemption</span>,
    [Deadlock](ch01.html#idm140183163376992)

</div>

<div data-type="indexdiv">

### O

  - <span data-type="index-term">object pool pattern</span>,
    [Pool](ch03.html#idm140183154163376)
  - <span data-type="index-term">once variable</span>
  - <span data-type="index-term">online resources</span>, [Online
    Resources](preface01.html#idm140183166962512)
  - <span data-type="index-term">or-channel</span>, [The
    or-channel](ch04.html#orchannel04)-[The
    or-channel](ch04.html#idm140183149049184)
  - <span data-type="index-term">or-done-channels</span>, [The
    or-done-channel](ch04.html#idm140183145447776)
  - <span data-type="index-term">order-independence</span>, [Fan-Out,
    Fan-In](ch04.html#idm140183146198688)
  - <span data-type="index-term">OS threads</span>, [The Difference
    Between Concurrency and Parallelism](ch02.html#idm140183157947376)

</div>

<div data-type="indexdiv">

### P

  - <span data-type="index-term">parallelism</span>, [The Difference
    Between Concurrency and Parallelism](ch02.html#parallel02)-[The
    Difference Between Concurrency and
    Parallelism](ch02.html#idm140183157695328), [How This Helps
    You](ch02.html#idm140183157618080)
  - <span data-type="index-term">parent cancellations</span>, [Timeouts
    and Cancellation](ch05.html#idm140183142149952)
  - <span data-type="index-term">patterns</span>
    (<span data-gentext="see">see</span> concurrency patterns)
  - <span data-type="index-term">pipelines</span>
      - <span data-type="index-term">batch processing</span>,
        [Pipelines](ch04.html#idm140183147917792)
      - <span data-type="index-term">benefits of</span>,
        [Pipelines](ch04.html#idm140183148382512)
      - <span data-type="index-term">best practices for
        constructing</span>, [Best Practices for Constructing
        Pipelines](ch04.html#Pbest04)-[Best Practices for Constructing
        Pipelines](ch04.html#idm140183147104784)
      - <span data-type="index-term">definition of term</span>,
        [Pipelines](ch04.html#idm140183148379680)
      - <span data-type="index-term">empty interfaces and</span>, [Some
        Handy Generators](ch04.html#idm140183146567712)
      - <span data-type="index-term">handy generators</span>, [Some
        Handy Generators](ch04.html#Pgener04)-[Some Handy
        Generators](ch04.html#idm140183146211328)
      - <span data-type="index-term">properties of</span>,
        [Pipelines](ch04.html#idm140183148068416)
      - <span data-type="index-term">stages</span>,
        [Pipelines](ch04.html#idm140183148378320)
      - <span data-type="index-term">stream processing</span>,
        [Pipelines](ch04.html#idm140183147915600)
  - <span data-type="index-term">pool pattern</span>,
    [Pool](ch03.html#pool03)-[Pool](ch03.html#idm140183153253072)
  - <span data-type="index-term">pprof profiler</span>,
    [pprof](app01.html#idm140183133201712)
  - <span data-type="index-term">preemptability</span>, [Timeouts and
    Cancellation](ch05.html#prempt05)-[Timeouts and
    Cancellation](ch05.html#idm140183139493120)
  - <span data-type="index-term">primitives</span>
    (<span data-gentext="see">see</span> concurrency primitives)
  - <span data-type="index-term">process calculi</span>, [What Is
    CSP?](ch02.html#idm140183157687072)

</div>

<div data-type="indexdiv">

### Q

  - <span data-type="index-term">questions and comments</span>, [How to
    Contact Us](preface01.html#idm140183166994368)
  - <span data-type="index-term">queuing</span>
      - <span data-type="index-term">benefits and drawbacks of</span>,
        [Queuing](ch04.html#idm140183144435840)
      - <span data-type="index-term">best use of</span>,
        [Queuing](ch04.html#idm140183143902096)
      - <span data-type="index-term">blocking and</span>,
        [Queuing](ch04.html#idm140183144120032)
      - <span data-type="index-term">chunking and</span>,
        [Queuing](ch04.html#idm140183143919872)
      - <span data-type="index-term">decoupling ability of</span>,
        [Queuing](ch04.html#idm140183144017632)
      - <span data-type="index-term">Little’s law</span>,
        [Queuing](ch04.html#idm140183143899024)
      - <span data-type="index-term">negative feedback loops</span>,
        [Queuing](ch04.html#idm140183143913168)
      - <span data-type="index-term">performance concerns and</span>,
        [Queuing](ch04.html#idm140183144431088)
      - <span data-type="index-term">using</span>,
        [Queuing](ch04.html#idm140183144015536)

</div>

<div data-type="indexdiv">

### R

  - <span data-type="index-term">race conditions</span>, [Race
    Conditions](ch01.html#race01)-[Race
    Conditions](ch01.html#idm140183163955264), [Race
    Detection](app01.html#idm140183133316096)
  - <span data-type="index-term">rate limiting</span>
      - <span data-type="index-term">benefits of</span>, [Rate
        Limiting](ch05.html#idm140183136680128)
      - <span data-type="index-term">definition of term</span>, [Rate
        Limiting](ch05.html#idm140183137892144)
      - <span data-type="index-term">implementing</span>, [Rate
        Limiting](ch05.html#RLimp05)-[Rate
        Limiting](ch05.html#idm140183136197072)
      - <span data-type="index-term">purpose of</span>, [Rate
        Limiting](ch05.html#idm140183137890496)
      - <span data-type="index-term">single vs. aggregate</span>, [Rate
        Limiting](ch05.html#RLsingle05)-[Rate
        Limiting](ch05.html#idm140183135460160)
  - <span data-type="index-term">replicated requests</span>, [Timeouts
    and Cancellation](ch05.html#idm140183142146864), [Replicated
    Requests](ch05.html#repreq05)-[Replicated
    Requests](ch05.html#idm140183137896656)
  - <span data-type="index-term">resources</span>, [Online
    Resources](preface01.html#idm140183166960480)
  - <span data-type="index-term">runtime/pprof package</span>,
    [pprof](app01.html#idm140183133198480)
  - <span data-type="index-term">RWMutex and Mutex</span>, [Mutex and
    RWMutex](ch03.html#rwmutex03)-[Mutex and
    RWMutex](ch03.html#idm140183154668576)

</div>

<div data-type="indexdiv">

### S

  - <span data-type="index-term">scaling concurrent operations</span>
      - <span data-type="index-term">dynamic scaling</span>, [How This
        Helps You](ch02.html#idm140183157615136)
      - <span data-type="index-term">error propagation</span>, [Error
        Propagation](ch05.html#SCOerror05)-[Error
        Propagation](ch05.html#idm140183142181680)
      - <span data-type="index-term">healing unhealthy
        goroutines</span>, [Healing Unhealthy
        Goroutines](ch05.html#SCOheal05)-[Healing Unhealthy
        Goroutines](ch05.html#idm140183134047296)
      - <span data-type="index-term">heartbeats</span>,
        [Heartbeats](ch05.html#SCOheart05)-[Heartbeats](ch05.html#idm140183137550208)
      - <span data-type="index-term">horizontal scaling</span>, [Moore’s
        Law, Web Scale, and the Mess We’re
        In](ch01.html#idm140183167015568)
      - <span data-type="index-term">rate limiting</span>, [Rate
        Limiting](ch05.html#SCOrate05)-[Rate
        Limiting](ch05.html#idm140183135461136)
      - <span data-type="index-term">replicated requests</span>,
        [Replicated Requests](ch05.html#SCOrep05)-[Replicated
        Requests](ch05.html#idm140183137897632)
      - <span data-type="index-term">timeouts and cancellation</span>,
        [Timeouts and Cancellation](ch05.html#SCOtime05)-[Timeouts and
        Cancellation](ch05.html#idm140183142373088)
  - <span data-type="index-term">select statement</span>, [How This
    Helps You](ch02.html#idm140183157611424), [The select
    Statement](ch03.html#select03)-[The select
    Statement](ch03.html#idm140183150991248)
  - <span data-type="index-term">shared state</span>, [Timeouts and
    Cancellation](ch05.html#idm140183139491008)
  - <span data-type="index-term">Signal method</span>,
    [Cond](ch03.html#idm140183155262768)
  - <span data-type="index-term">spigot algorithms</span>, [Moore’s Law,
    Web Scale, and the Mess We’re In](ch01.html#idm140183167019536)
  - <span data-type="index-term">stale data</span>, [Timeouts and
    Cancellation](ch05.html#idm140183142166272)
  - <span data-type="index-term">stalling joins</span>, [Stealing Tasks
    or Continuations?](ch06.html#idm140183133623600)
  - <span data-type="index-term">starvation</span>,
    [Starvation](ch01.html#starv01)-[Starvation](ch01.html#idm140183158021984)
  - <span data-type="index-term">stream processing</span>,
    [Pipelines](ch04.html#idm140183147916272)
  - <span data-type="index-term">Sutter, Herb</span>, [Moore’s Law, Web
    Scale, and the Mess We’re In](ch01.html#idm140183162923728)
  - <span data-type="index-term">sync package</span>
      - <span data-type="index-term">Cond</span>,
        [Cond](ch03.html#SPcond03)-[Cond](ch03.html#idm140183154953584)
      - <span data-type="index-term">memory access
        synchronization</span>, [Memory Access
        Synchronization](ch01.html#SPmomory01)-[Memory Access
        Synchronization](ch01.html#idm140183163637200),
        [Starvation](ch01.html#idm140183158027456), [What Is
        CSP?](ch02.html#idm140183157650976), [The sync
        Package](ch03.html#idm140183156318464)
      - <span data-type="index-term">Mutex and RWMutex</span>, [Mutex
        and RWMutex](ch03.html#SPmutex03)-[Mutex and
        RWMutex](ch03.html#idm140183154669520)
      - <span data-type="index-term">once variable</span>,
        [Once](ch03.html#Sonce03)-[Pool](ch03.html#idm140183154160480)
      - <span data-type="index-term">Pool</span>,
        [Pool](ch03.html#Spool03)-[Pool](ch03.html#idm140183153254048)
      - <span data-type="index-term">WaitGroup</span>,
        [WaitGroup](ch03.html#idm140183156309104)
  - <span data-type="index-term">system saturation</span>, [Timeouts and
    Cancellation](ch05.html#idm140183142173184)

</div>

<div data-type="indexdiv">

### T

  - <span data-type="index-term">tee-channels</span>, [The
    tee-channel](ch04.html#idm140183145024880)
  - <span data-type="index-term">thread pools</span>, [Simplicity in the
    Face of Complexity](ch01.html#idm140183157811776), [The Difference
    Between Concurrency and Parallelism](ch02.html#idm140183157943680),
    [How This Helps You](ch02.html#idm140183157636656)
  - <span data-type="index-term">timeouts and cancellations</span>
      - <span data-type="index-term">duplicated messages and</span>,
        [Timeouts and Cancellation](ch05.html#TACduplicate05)-[Timeouts
        and Cancellation](ch05.html#idm140183142371168)
      - <span data-type="index-term">premptability and</span>, [Timeouts
        and Cancellation](ch05.html#TACpreempt05)-[Timeouts and
        Cancellation](ch05.html#idm140183139493968)
      - <span data-type="index-term">role of cancellations</span>,
        [Timeouts and Cancellation](ch05.html#idm140183142155872)
      - <span data-type="index-term">role of timeouts</span>, [Timeouts
        and Cancellation](ch05.html#idm140183142176736)
      - <span data-type="index-term">shared state modifications</span>,
        [Timeouts and Cancellation](ch05.html#idm140183139491920)
  - <span data-type="index-term">tools and commands</span>
      - <span data-type="index-term">goroutine errors</span>, [Anatomy
        of a Goroutine Error](app01.html#idm140183133364944)
      - <span data-type="index-term">pprof profiler</span>,
        [pprof](app01.html#idm140183133202560)
      - <span data-type="index-term">race detection</span>, [Race
        Detection](app01.html#idm140183133317072)
  - <span data-type="index-term">typographical conventions</span>,
    [Conventions Used in This Book](preface01.html#idm140183166948368)

</div>

<div data-type="indexdiv">

### U

  - <span data-type="index-term">uninterruptible</span>,
    [Atomicity](ch01.html#idm140183163941312)

</div>

<div data-type="indexdiv">

### W

  - <span data-type="index-term">wait for condition</span>,
    [Deadlock](ch01.html#idm140183163391392)
  - <span data-type="index-term">WaitGroup</span>,
    [WaitGroup](ch03.html#idm140183156308096)
  - <span data-type="index-term">web scale</span>, [Moore’s Law, Web
    Scale, and the Mess We’re In](ch01.html#idm140183162927216)
  - <span data-type="index-term">work cancellation</span>, [Preventing
    Goroutine Leaks](ch04.html#idm140183150175664), [Timeouts and
    Cancellation](ch05.html#idm140183142178656)
      - (<span data-gentext="see">see also</span> timeouts and
        cancellations)
  - <span data-type="index-term">work stealing strategy</span>, [Work
    Stealing](ch06.html#wrksteal05)-[Stealing Tasks or
    Continuations?](ch06.html#idm140183133375552)
