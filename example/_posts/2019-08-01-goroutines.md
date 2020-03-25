---
layout: post
title: Goroutines
description: >
  A page showing Hydejack-specific markdown content.
image: /assets/img/blog/example-content-iii.jpg
noindex: true
---

#### # Goroutines
>A goroutine is a lightweight thread managed by the Go runtime.
>go f(x, y, z)
starts a new goroutine running
f(x, y, z)
The evaluation of f, x, y, and z happens in the current goroutine and the execution of f happens in the new goroutine.
Goroutines run in the same address space, so access to shared memory must be synchronized

Suppose we have a function call f(s). Here’s how we’d call that in the usual way, running it synchronously.
``` 
f("direct")
```
To invoke this function in a goroutine, use go f(s). 
```
go f("from goroutine")
```
This new goroutine will execute concurrently with the calling one.

You can also start a goroutine for an anonymous function call.
```
go func(msg string) {
        fmt.Println(msg)
    }("going")
```

Two important properties of goroutine:

- When a new Goroutine is started, the goroutine call returns immediately. Unlike functions, the control does not wait for the Goroutine to finish executing. The control returns immediately to the next line of code after the Goroutine call and any return values from the Goroutine are ignored.
- The main Goroutine should be running for any other Goroutines to run. If the main Goroutine terminates then the program will be terminated and no other Goroutine will run.

