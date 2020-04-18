---
layout: post
title: Beauty of Go
description: >
  Exploring the beauty of Go
image: /assets/img/beauty.jpg
noindex: true
---



### Beauty of Go

### Introduction
Go is an open source language, created at Google by Robert Griesemer, Rob Pike, and Ken Thompson. 

> “Go will be the server language of the future.” — Tobias Lütke, Shopify

Go was designed with an eye on felicity of programming, speed of compilation, orthogonality of concepts, and the need to support features such as concurrency and garbage collection.

### Why was Go needed?

The authors mention that the primary motive for designing a new language was to solve software engineering issues at Google. They also mention that Go was actually developed as an alternative to C++.

Rob Pike mentions the purpose for the Go programming language:

> “Go’s purpose is therefore not to do research into programming language design; it is to improve the working environment for its designers and their coworkers. Go is more about software engineering than programming language research. Or to rephrase, it is about language design in the service of software engineering.”

Issues that were plaguing the software engineering horizon at Google were:

 1. slow builds(builds would sometime take as long as an hour to complete)
 1. uncontrolled dependencies
 1. each programmer using a different subset of the language
 1. poor program understanding (code hard to read, poorly documented, and so on)
 1. duplication of effort
 1. cost of updates
 1. version skew
 1. difficulty of writing automatic tools
 1. cross-language builds
 
For Go to succeed, Go must solve these problems:

1. Go must work at scale, for large teams of programmers working on them, for programs with large numbers of dependencies.
 1. Go must be familiar, roughly C-like. Google needs to get programmers productive quickly in Go, means that the language cannot be too radical.
 1. Go must be modern. It should have features like concurrency so that programs can make efficient use of multi core machines. It should have built-in networking and web server libraries so that it aids modern development.
 
### Target Audience

Go is a systems programming language. Go really shines for stuff such as cloud systems (web servers, caches), microservices, distributed systems (due to concurrency support).

### Strengths

#### #1 Go is easy to learn
>You can learn most of Go's syntax in a couple of hours with the "Tour of Go".Because of Go's simplicity, Go programs are very readable.Go makes it easier to write correct, clear and efficient code.Go has marvelous documentation, and most of the advanced topics have already been covered on their blog: The Go Programming Language Blog.

Go intentionally leaves out many features of modern OOP languages:

1. No classes. Every thing is divided into packages only. Go has only structs instead of classes.
1. Does not support inheritance. That will make code easy to modify. In other languages like Java/Python, if the class ABC inherits class XYZ and you make some changes in class XYZ, then that may produce some side effects in other classes that inherit XYZ. By removing inheritance, Go makes it easy to understand the code also (as there is no super class to look at while looking at a piece of code).
1. No constructors.
1. No annotations.
1. No generics.
1. No exceptions.


####  #2 Portable
> Go is a compiled language.Since the code gets directly compiled to machine code, therefore, the binaries become portable. Portability here means that you can pick up the binary from your machine (let’s say Linux, x86–64) and directly run that on your server (if your server is also running Linux on a x86–64 architecture).

> An executable Go program typically consists of a single standalone binary, with no separate dynamic libraries or virtual machines, (meaning that any shared operating system libraries your program needs are included in the binary at the time of the compilation. They are not dynamically linked at the time of running the program) which can be directly deployed.

Compared specifically to Java, Go “compiles to small statically linked binaries” which can run directly on a server with no dependencies – rather than having to be executed on the JVM “within a servlet container”.

This has a huge benefit for deployment of your programs on multiple machines in a data center. If you have 100 machines in your data center, you can simply ‘scp’ your program binary to all of them, as long as the binary is compiled for the same OS and instruction set architecture your machines run on. You don’t need to care about which version of Linux they are running. There is no need for checking/managing dependencies. The binaries simply run and all your services are up 


####  #3 Statically typed with no implicit conversions
> Go is statically typed. This means that you need to declare types for all your variables and your function arguments (and return variables) at compile time. Although this may sound inconvenient, this is a great advantage since a lot of errors will be found at compile time itself. This factor plays a very big role when your team size increases, since declared types make functions and libraries more readable and more easier to understand.

#### #4 Fast Compilation
> Go code compiles really fast, so you don’t need to keep waiting for your code to compile.

#### #5 High Execution Speed
>Go compiles to a native executable.ie Go code gets directly compiled to machine code, depending upon the OS (Linux/Windows/Mac) and the CPU instruction set architecture (x86, x86–64, arm etc) of the machine the code is being compiled upon. So, it runs really fast.

#### #6 Code separation and efficient management of dependencies.
 > Programs are constructed from packages that offer clear code separation and allow efficient management of dependencies. The package mechanism is perhaps the single most well-designed feature of the language, and certainly one of the most overlooked.
 
#### #7 Structurally typed interfaces
> Interfaces enable loosely coupled systems.Structurally typed interfaces provide runtime polymorphism through dynamic dispatch.A type implements an interface by implementing its methods. No explicit declaration is required.This is checked by the compiler automatically at compile time.

This means that a part of your code can just rely on an interface type and doesn’t really care about who implements the interface or how the interface is actually implemented. Your main/controller function can then supply a dependency which satisfies the interface (implements all the functions in the interface) to that code. This also enables a really clean architecture for unit testing (through dependency injection). Now, your test code can just inject a mock implementation of the interface required by the code to be able to test if it’s doing its job correctly or not.

![](assets/img/comp.png)


#### #8 Easy concurrent programming with goroutines and channels
> Concurrency, by its design, enables you to efficiently use your CPU horsepower. Even if your processor just has 1 core, concurrency’s design enables you to use that one core efficiently. 

> "Goroutines are probably the best feature of Go. They're lightweight computation threads, distinct from operating system threads.The Go runtime allows you to run hundreds of thousands of concurrent goroutines on a machine. A Goroutine is a lightweight thread of execution. The Go runtime multiplexes those goroutines over operating system threads. That means that multiple goroutines can run concurrently on a single OS thread. The Go runtime has a scheduler whose job is to schedule these goroutines for execution.

There are two benefits of this approach:

1.  A Goroutine when initialized has a stack of 4 KB. This is really tiny as compared to a stack of an OS thread, which is generally 1 MB. This number matters when you need to have hundreds of thousands of different goroutines running concurrently. If you would run more than thousands of OS threads in parallel, the RAM obviously will become a bottleneck.

1.  Go could have followed the same model as other languages like Java, which support the same concept of threads as OS threads. But in that case, the cost of a context switch between OS threads is much larger than the cost of a context switch between different goroutines.

>When a Go program executes what looks like a blocking I/O operation, the Go runtime actually suspends the goroutine and resumes it when an event indicates that some result is available. In the meantime other goroutines have been scheduled for execution. We therefore have the scalability benefits of asynchronous programming with a synchronous programming model.

>Channels are how goroutines should communicate: they provide a convenient programming model to send and receive data between goroutines without having to rely on fragile low level synchronization primitives. 

#### #9 Great standard libraries
> The Go standard library is really great, particularly for everything related to network protocols or API development: http client and server, crypto, archive formats, compressions, sending email, etc.
Some of them are:

1. net/http-rovides HTTP client and server implementations

1.database/sql-For interaction with SQL databases

1. encoding/json-JSON is treated as a first class member of the standard language :)

1. html/templates-HTML templating library

1. io/ioutil-Implements I/O utility functions

#### #10 Go is garbage collected
> Unlike C, you don’t need to remember to free up pointers or worry about dangling pointers in Go. The garbage collector automatically does this job. Garbage collection protects against memory leaks. The collection has very low latency. 

#### #11 No exceptions, handle errors yourself
> Although, it can also be considered as a drawback of Go, it is still good in a way that it forces developers to handle basic errors like ‘couldn’t open file’ etc rather than letting them wrap up all of their code in a try catch block. This also puts pressure on developers to actually think about what needs to be done to handle these failure scenarios.

#### #12 Language defined source code format
> Some of the most heated debates has happened around the definition of a code format for the team. Go solves this by defining a canonical format for Go code. The gofmt tool reformats your code and has no options. You have only one style guide that everyone follows.

#### #13 Amazing tooling
>One of the best aspects about Go is its tooling. Some of them are:

1.  Gofmt: It automatically formats and indents your code so that your code looks like the same as every Go developer on the planet. This has a huge effect on code readability.

1.  Go run: This compiles your code and runs it, both :). So even though Go needs to be compiled, this tool makes you feel like it’s an interpreted language since it just compiles your code so fast that you don’t even feel when the code got compiled.

1. Go get: This downloads the library from GitHub and copies it to your GoPath so that you can import the library in your project

1. Godoc: Go has automatically generated documentation with testable examples.
Godoc parses your Go source code including comments and produces its documentation in HTML or plain text format. Through godoc’s web interface, you can then see documentation tightly coupled with the code it documents. You can navigate from a function’s documentation to its implementation with one click.

1. Go test: Go comes with a great test framework in its standard library. It has support for parallel testing, benchmarks, and contains a lot of utilities to easily test network clients and servers.

#### #14 Static code analysis

> Go heavily relies on static code analysis. Examples include godoc for documentation, gofmt for code formatting, golint for code style linting, and many others.
Those tools are commonly implemented as stand-alone command line applications and integrate easily with any coding environment.
Static code analysis isn’t actually something new to modern programming, but Go sort of brings it to the absolute. Also, it gives you a feeling of safety, as though someone is covering your back.

#### #15 Race condition detection
> Race conditions happen when several concurrent operations finish in an unpredicted order. It might lead to a huge number of bugs, which are particularly hard to chase down. 
Go has quite a powerful tool to hunt those race conditions down. It is fully integrated into Go’s toolchain.

#### #16 Reflection
Code reflection is essentially an ability to sneak under the hood and access different kinds of meta-information about your language constructs, such as variables or functions.Given that Go is a statically typed language, it’s exposed to a number of various limitations when it comes to more loosely typed abstract programming.
> However, there are still cases in which you can’t possibly know what sort of data you are facing.In order to pull that off, you need a tool to examine all the data in runtime that acts differently depending on its type and structure. Reflection to rescue! Go has a first-class reflect package to enable your code to be as dynamic as it would be in a language like Javascript.

Note that you only use it when there is no simpler way.

#### #17 Defer statement, to avoid forgetting to clean up
> The defer statement serves a purpose similar to finally in Java: execute some clean up code at the end of the current function, no matter how this function is exited. The interesting thing with defer is that it's not linked to a block of code, and can appear at any time. This allows the clean up code to be written as close as possible to the code that creates what needs to be cleaned up

#### #18  User defined types
> Go has first-class support for new types, i.e. types that take an existing type and give it a separate identity, distinct from the original one. 

### What's so special about Goroutines?

Most of the modern programming languages(like Java, Python etc.) are from the ’90s single threaded environment.Most of those programming languages supports multi-threading. But the real problem comes with concurrent execution, threading-locking, race conditions and deadlocks. Those things make it hard to create a multi-threading application on those languages.

On the other hand, Go was released in 2009 when multi-core processors were already available. That’s why Go is built with keeping concurrency in mind. Go has goroutines instead of threads. They consume almost 2KB memory from the heap. So, you can spin millions of goroutines at any time.

Unlike Java threads, which are blocking by nature, Goroutines are non-blocking. Goroutines is the combination of async approach used by JavaScript and the traditional multi-threading used by Java. 

For an example, creating new thread in Java is not memory efficient. As every thread consumes approx 1MB of the memory heap size and eventually if you start spinning thousands of threads, they will put tremendous pressure on the heap and will cause shut down due to out of memory. Also, if you want to communicate between two or more threads, it’s very difficult.

Other benefits are :

1. Goroutines have growable segmented stacks. That means they will use more memory only when needed.
1. Goroutines have a faster startup time than threads.
1. Goroutines come with built-in primitives to communicate safely between themselves (channels).
1. Goroutines allow you to avoid having to resort to mutex locking when sharing data structures.
1. Also, goroutines and OS threads do not have 1:1 mapping. A single goroutine can run on multiple threads. Goroutines are multiplexed into small number of OS threads.
