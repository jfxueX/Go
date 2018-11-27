# Preface

I just love new programming languages. Perhaps it's the inevitable familiarity 
and ennui with regard to existing languages and the frustration with existing 
tools, syntaxes, coding conventions, and performance. Maybe I'm just hunting for 
that one "language to rule them all". Whatever the reason, any time a new or 
experimental language is released, I have to dive right in.

This has been a golden age for new languages and language design. Think about 
it: the C language was released in the early 1970s—a time when resources were so 
scarce that verbosity, clarity, and syntactical logic were often eschewed for 
thrift. And most of the languages we use today were either originally written in 
this era or were directly influenced by those languages.

Since the late 1980s and early 1990s, there has been a slow flood of powerful 
new languages and paradigms—Perl, Python, Ruby, PHP, and JavaScript—have taken 
an expanding user base by storm and has become one of the most popular languages 
(up there with stalwarts such as C, C++, and Java). Multithreading, memory 
caching, and APIs have allowed multiple processes, dissonant languages, 
applications, and even separate operating systems to work in congress.

And while this is great, there's a niche that until very recently was largely 
unserved: powerful, compiled, cross-platform languages with concurrency support 
that are geared towards systems programmers.

Very few languages match these parameters. Sure, there have been lower-level 
languages that fulfill some of these characteristics. Erlang and Haskell fit the 
bill in terms of power and language design, but as functional languages they 
pose a learning barrier for systems programmers coming from a C/Java background. 
Objective-C and C\# are relatively easy, powerful, and have concurrency support—
but they're bound enough to a specific OS to make programming for other 
platforms arduous. The languages we just mentioned (Python, JavaScript, and so 
on)—while extremely popular—are largely interpreted languages, forcing 
performance into a secondary role. You can use most of them for systems 
programming, but in many ways it's the proverbial square peg in a round hole. So 
when Google announced Go in 2009, my interest was piqued. When I saw who was 
behind the project (more on that later), I was elated.  When I saw the language 
and its design in action, I was in heaven.

For the last few years I've been using Go to replace systems applications I'd 
previously written in C, Java, Perl, and Python. I couldn't be happier with the 
results. Implementing Go has improved these applications in almost every 
instance. The fact that it plays nicely with C is another huge selling point for 
systems programmers looking to dip their toes in Go's pool.

With some of the best minds in language design (and programming in general) 
behind it, Go has a bright future.

For years—decades, really—there have been less than a handful of options for 
writing servers and network interfaces. If you were tasked with writing one, you 
probably reached for C, C++, or Java. And while these certainly can handle the 
task, and while they all now support concurrency and parallelism in some way or 
another, they weren't designed for that.

Google brought together a team that included some giants of programming—Rob Pike 
and Ken Thompson of Bell Labs fame and Robert Griesemer, who worked on Google's 
JavaScript implementation V8—to design a modern, concurrent language with 
development ease at the forefront.

To do this, the team focused on some sore spots in the alternatives, which are 
as follows:


  - Dynamically typed languages have—in recent years—become incredibly
    popular. Go eschews the explicit, "cumbersome" type systems of Java
    or C++. Go uses type inference, which saves development time, but is
    still also strongly typed.
  - Concurrency, parallelism, pointers/memory access, and garbage
    collection are unwieldy in the aforementioned languages. Go lets
    these concepts be as easy or as complicated as you want or need them
    to be.
  - As a newer language, Go has a focus on multicore design that was a
    necessary afterthought in languages such as C++.
  - Go's compiler is super-fast; it's so fast that there are
    implementations of it that treat Go code as interpreted.
  - Although Google designed Go to be a systems language, it's versatile
    enough to be used in a myriad of ways. Certainly, the focus on
    advanced, cheap concurrency makes it ideal for network and systems
    programming.
  - Go is loose with syntax, but strict with usage. By this we mean that
    Go will let you get a little lazy with some lexer tokens, but you
    still have to produce fundamentally tight code. As Go provides a
    formatting tool that attempts to clarify your code, you can also
    spend less time on readability concerns as you're coding.

## What this book covers

[1]: ./ch01-an-introduction-to-concurrency-in-go.md
[2]: ./ch02-understanding-the-concurrency-model.md
[3]: ./ch03-developing-a-concurrent-strategy.md
[4]: ./ch04-data-integrity-in-an-application.md
[5]: ./ch05-locks-blocks-and-better-channels.md
[6]: ./ch06-c10k-a-non-blocking-web-server-in-go.md
[7]: ./ch07-performace-and-scalability.md
[8]: ./ch08-concurrent-application-architecture.md
[9]: ./ch09-logging-and-testing-concurrency-in-go.md
[10]: ./ch10-advanced-concurrency-and-best-pratices.md

[Chapter 1][1], *An Introduction to Concurrency in Go*, introduces goroutines 
and channels, and will compare the way Go handles concurrency with the approach 
other languages use. We'll build some basic concurrent applications utilizing 
these new concepts.

[Chapter 2][2], *Understanding the Concurrency Model*, focuses on resource 
allocation, sharing memory (and when not to), and data. We will look at channels 
and channels of channels as well as explain exactly how Go manages concurrency 
internally.

[Chapter 3][3], *Developing a Concurrent Strategy*, discusses approach methods 
for designing applications to best use concurrent tools in Go. We'll look at 
some available third-party packages that can play a role in your strategy.

[Chapter 4][4], *Data Integrity in an Application*, looks at ensuring that 
delegation of goroutines and channels maintain the state in single thread and 
multithread applications.

[Chapter 5][5], *Locks, Blocks, and Better Channels*, looks at how Go can avoid 
dead locks out of the box, and when and where they can still occur despite Go's 
language design.

[Chapter 6][6], *C10K – A Non-blocking Web Server in Go*, tackles one of the 
Internet's most famous and esteemed challenges and attempt to solve it with core 
Go packages. We'll then refine the product and test it against common 
benchmarking tools.

[Chapter 7][7], *Performance and Scalability*, focuses on squeezing the most out 
of your concurrent Go code, best utilizing resources and accounting for and 
mitigating third-party software's impact on your own. We'll add some additional 
functionality to our web server and talk about other ways in which we can use 
these packages.

[Chapter 8][8], *Concurrent Application Architecture*, focuses on when and where 
to implement concurrent patterns, when and how to utilize parallelism to take 
advantage of advanced hardware, and how to ensure data consistency.

[Chapter 9][9], *Logging and Testing Concurrency in Go*, focuses on OS-specific 
methods for testing and deploying your application. We'll also look at Go's 
relationship with various code repositories.

[Chapter 10][10], *Advanced Concurrency and Best Practices*, looks at more 
complicated and advanced techniques including duplicating concurrent features 
not available in Go's core.

## What you need for this book

To work along with this book's examples, you'll need a computer running Windows, 
OS X, or quite a few Linux variants that support Go. For this book, our Linux 
examples and notes reference Ubuntu.

If you do not already have Go 1.3 or newer installed, you will need to get it 
either from the binaries download page on <http://golang.org/> or through your 
operating system's package manager.

To use all of the examples in this book, you'll also need to have the following 
software installed:

  - MySQL (<http://dev.mysql.com/downloads/>)
  - Couchbase (<http://www.couchbase.com/download>)

Your choice of IDE is a matter of personal preference, as anyone who's worked 
with developers can attest. That said, there are a few that lend themselves 
better to some languages than others and a couple that have good support for Go. 
This author uses Sublime Text, which plays very nice with Go, is lightweight, 
and allows you to build directly from within the IDE itself. Anywhere you see 
screenshots of code, it will be from within Sublime Text.

And while there's a good amount of baked-in support for Go code, there's also a 
nice plugin collection for Sublime Text called GoSublime, available at 
<https://github.com/DisposaBoy/GoSublime>.

Sublime Text isn't free, but there is a free evaluation version available that 
has no time limit. It's available in Windows, OS X, and Linux variants at 
<http://www.sublimetext.com/>.


## Who this book is for

If you are a systems or network programmer with some knowledge of Go and 
concurrency, but would like to know about the implementation of concurrent 
systems written in Go this is the book for you. The goal of this book is to 
enable you to write high-performance, scalable, resource-thrifty systems and 
network applications in Go.

In this book, we'll write a number of basic and somewhat less - basic network 
and systems applications. It's assumed that you've worked with these types of 
applications before. If you haven't, some extracurricular study may be warranted 
to be able to fully digest this content.


## Conventions

In this book, you will find a number of styles of text that distinguish between 
different kinds of information. Here are some examples of these styles, and an 
explanation of their meaning.

Code words in text, database table names, folder names, filenames, file 
extensions, pathnames, dummy URLs, user input, and Twitter handles are shown as 
follows: "The `setProxy` function is called after every request, and you can see 
it as the first line in our handler."

A block of code is set as follows:

```go
package main

import
(
    "net/http"
    "html/template"
    "time"
    "regexp"
    "fmt"
    "io/ioutil"
    "database/sql"
    "log"
    "runtime"
    _ "github.com/go-sql-driver/mysql"
)
```

When we wish to draw your attention to a particular part of a code block, the 
relevant lines or items are set in bold:

```go
package main

import (
  "fmt"
)

func stringReturn(text string) string {
  return text
}

func main() {
  myText := stringReturn("Here be the code")
  fmt.Println(myText)
}
```

Any command-line input or output is written as follows:

```bash
go get github.com/go-sql-driver/mysql
```

**New terms** and **important words** are shown in bold. Words that you see on 
the screen, in menus or dialog boxes for example, appear in the text like this: 
"If you upload a file by dragging it to the **Drop files here to upload** box, 
within a few seconds you'll see that the file is noted as changed in the web 
interface."


### Note

Warnings or important notes appear in a box like this.

### Tip

Tips and tricks appear like this.

