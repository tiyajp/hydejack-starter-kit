---
layout: post
title: Goroutines
description: >
  An Introduction to Goroutines.
image: /assets/img/goroutine.png
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

Question: Why does this program exit without starting the worker function? How to make it work properly?

```
package main

import (
    "fmt"
    "time"
)

func worker(done chan bool) {
    fmt.Print("working...")
    fmt.Println("done")
    done <- true
}

func main() {
    done := make(chan bool, 1)
    go worker(done)
    fmt.Println("Exiting from main()")
}
```

Output:

```
Exiting from main()
```


>In the above case, the main goroutine spawns another goroutine of worker function. Hence when we execute the above program, there are two goroutines running concurrently.Goroutines are scheduled cooperatively. Hence when the main goroutine starts executing, go scheduler do not pass control to the worker goroutine until the main goroutine does not execute completely. Unfortunately, when the main goroutine is done with execution, the program terminates immediately and scheduler did not get time to schedule worker goroutine.

Solution:

```
package main

import (
    "fmt"
    "time"
)

func worker(status string ) {
    fmt.Println(status)  
}

func main() {

    done := make(chan bool, 1)
    go worker("Working")
    time.Sleep(10 * time.Millisecond)
    fmt.Println("Exiting from main()")
}
```

Output:

```
Working
Exiting from main()
```

- Using time.Sleep()
Before main goroutine pass control to the last line of code, we pass control to worker goroutine using time.Sleep(10 * time.Millisecond) call. In this case, the main goroutine sleeps for 10 milli-seconds and won’t be scheduled again for another 10 milliseconds.
