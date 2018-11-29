# Chapter 3. Developing a Concurrent Strategy

In the previous chapter, we looked at the concurrency model that Go relies on to 
make your life as a developer easier. We also saw a visual representation of 
parallelism and concurrency. These help us to understand the differences and 
overlaps between serialized, concurrent, and parallel applications.

However, the most critical part of any concurrent application is not the 
concurrency itself but communication and coordination between the concurrent 
processes.

In this chapter, we'll look at creating a plan for an application that heavily 
factors communication between processes and how a lack of coordination can lead 
to significant issues with consistency. We'll look at ways we can visualize our 
concurrent strategy on paper so that we're better equipped to anticipate 
potential problems.

## Applying efficiency in complex concurrency

When designing applications, we often eschew complex patterns for simplicity, 
with the assumption that simple systems are often the fastest and most 
efficient. It seems only logical that a machine with fewer moving parts will be 
more efficient than one with more.

The paradox here, as it applies to concurrency, is that adding redundancy and 
significantly more movable parts often leads to a more efficient application. If 
we consider concurrent schemes, such as goroutines, to be infinitely scalable 
resources, employing more should always result in some form of efficiency 
benefit. This applies not just to parallel concurrency but to single core 
concurrency as well.

If you find yourself designing an application that utilizes concurrency at the 
cost of efficiency, speed, and consistency, you should ask yourself whether the 
application truly needs concurrency at all.

When we talk about efficiency, we aren't just dealing with speed. Efficiency 
should also weigh the CPU and memory overhead and the cost to ensure data 
consistency.

For example, should an application marginally benefit from concurrency but 
require an elaborate and/or computationally expensive process to guarantee data 
consistency, it's worth re-evaluating the strategy entirely.

Keeping your data reliable and up to date should be paramount; while having 
unreliable data may not always have a devastating effect, it will certainly 
compromise the reliability of your application.

