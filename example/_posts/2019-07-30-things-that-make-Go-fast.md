---
layout: post
title: Things that make Go fast
description: >
  Summary of the important concepts from Dave Cheney's talk at Gocon, 2014.
image: /assets/img/speed.jpg
noindex: true
---

### Things that make Go fast

One of the design goals of Go was fast compilation. But How is it achieved?
Dave Cheney's talk at Gocon, a fantastic Go conference held semi-annually in Tokyo, Japan, 2014 covers 5 things that makes Go fast.
### #1 Go’s efficient treatment and storage of values.
```
var gocon int32 = 2014
```
This is an example of a value in Go. When compiled, gocon consumes exactly four bytes of memory.
Let’s compare Go with some other languages:

> Due to the overhead of the way Python represents variables, storing the same value using Python consumes six times more memory.This extra memory is used by Python to track type information, do reference counting, etc

> Similar to Go, the Java int type consumes 4 bytes of memory to store this value.
However, to use this value in a collection like a List or Map, the compiler must convert it into an Integer object. So an integer in Java frequently looks more like this and consumes between 16 and 24 bytes of memory.

```
type Location struct{
// 8 bytes per float64
// 24 bytes in total
    X,Y,Z float64
}
// Locations consume 24*1000 bytes
var Locations [1000]Locations
```
This is a Location type which holds the location of some object in three dimensional space. It is written in Go, so each Location consumes exactly 24 bytes of storage.

We can use this type to construct an array type of 1,000 Locations, which consumes exactly 24,000 bytes of memory.

> Inside the array, the Location structures are stored sequentially, rather than as pointers to 1,000 Location structures stored randomly.

This is important because now all 1,000 Location structures are in the cache in sequence, packed tightly together.

> Go lets you create compact data structures, avoiding unnecessary indirection.

>Compact data structures utilise the cache better.

>Better cache utilisation leads to better performance.

### #2 Inlining

Function calls are not free.

Three things happen when a function is called:

1. A new stack frame is created, and the return address of the caller recorded.

1. Any registers which may be overwritten during the function call are saved to the stack.

1. The processor computes the address of the function and executes a branch to that new address.

Because function calls are very common operations, CPU designers have worked hard to optimise this procedure, but they cannot eliminate the overhead.Depending on what the function does, this overhead may be trivial or significant.

A solution to reducing function call overhead is an optimisation technique called Inlining.

> The Go compiler inlines a function by treating the body of the function as if it were part of the caller.

Inlining has a cost; it increases binary size.

>It only makes sense to inline when the overhead of calling a function is large relative to the work the function does, so only simple functions are candidates for inlining.

Complicated functions are usually not dominated by the overhead of calling them and are therefore not inlined.

Inlining isn’t exclusive to Go. Almost every compiled or JITed language performs this optimisation. 

Having the source of the function enables other optimizations like Dead code elimination, thereby saves compiling or running any of the expensive code that is unreachable.
### #3 Escape Analysis

> Mandatory garbage collection makes Go a simpler and safer language.Memory allocated on the heap comes at a cost. It is a debt that costs CPU time every time the GC runs until that memory is freed.

Unlike C, which forces you to choose if a value will be stored on the heap, via malloc, or on the stack, by declaring it inside the scope of the function, Go implements an optimisation called "escape analysis"

>Escape analysis determines whether any references to a value escape the function in which the value is declared.

>If no references escape, the value may be safely stored on the stack.

>Values stored on the stack do not need to be allocated or freed.
 
If a declared variable is referenced only inside that particular function, the compiler will arrange to store the variable on the stack, rather than the heap.
There is no need to garbage collect that variable, it is automatically freed when function returns.

Go’s optimisations are always enabled by default. You can see the compiler’s escape analysis and inlining decisions with the -gcflags=-m switch.

> Because escape analysis is performed at compile time, not run time, stack allocation will always be faster than heap allocation, no matter how efficient your garbage collector is.

### #4 Goroutines
In a time-sharing system the operating systems must constantly switch the attention of the CPU between these processes by recording the state of the current process, then restoring the state of another.

This is called process switching. It has costs.

This lead to the development of threads, which are conceptually the same as processes, but share the same memory space.

> As threads share address space, they are lighter than processes so are faster to create and faster to switch between.

>Goroutines take the idea of threads a step further.

> Goroutines are cooperatively scheduled, rather than relying on the kernel to manage their time sharing.

> The switch between goroutines only happens at well defined points, when an explicit call is made to the Go runtime scheduler.

The compiler knows the registers which are in use and saves them automatically.
> While goroutines are cooperatively scheduled, this scheduling is handled for you by the runtime.

Places where Goroutines may yield to others are:

1. Channel send and receive operations, if those operations would block.
1. The Go statement, although there is no guarantee that new goroutine will be scheduled immediately.
1. Blocking syscalls like file and network operations.
1. After being stopped for a garbage collection cycle.

> Goroutines reduce the overhead of managing many, sometimes hundreds of thousands of concurrent threads of execution.
 
### #5 Copying stacks

Traditionally inside the address space of a process, the heap is at the bottom of memory, just above the program (text) and grows upwards.

The stack is located at the top of the virtual address space, and grows downwards.

Because the heap and stack overwriting each other would be catastrophic, the operating system usually arranges to place an area of unwritable memory between the stack and the heap to ensure that if they did collide, the program will abort.

This is called a guard page, and effectively limits the stack size of a process, usually in the order of several megabytes.

We’ve discussed that threads share the same address space, so for each thread, it must have its own stack.

> Because it is hard to predict the stack requirements of a particular thread, a large amount of memory is reserved for each thread’s stack along with a guard page.

> The downside is that as the number of threads in your program increases, the amount of available address space is reduced.

> Instead of using guard pages, the Go compiler inserts a check as part of every function call to check if there is sufficient stack for the function to run. If there is not, the runtime can allocate more stack space.

> Because of this check, a goroutines initial stack can be made much smaller, which in turn permits Go programmers to treat goroutines as cheap resources.

> Instead of adding and removing additional stack segments, if the stack of a goroutine is too small, a new, larger, stack will be allocated.

> The old stack’s contents are copied to the new stack, then the goroutine continues with its new larger stack.
