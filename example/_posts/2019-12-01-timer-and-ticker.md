---
layout: post
title: Timer and Ticker
description: >
  An Introduction to Timer and Ticker
image: /assets/img/blog/example-content-iii.jpg
noindex: true
---

#### # Timer and Ticker
##### Timer

>Timers represent a single event in the future. You tell the timer how long you want to wait, and it provides a channel that will be notified at that time.

When the Timer expires, the current time will be sent on C, unless the Timer was created by AfterFunc.
```
type Timer struct {
    C <-chan Time
    // contains filtered or unexported fields
}
```

```
package main

import (
    "fmt"
    "time"
)

func main() {

    timer1 := time.NewTimer(2 * time.Second)

    <-timer1.C
    fmt.Println("Timer 1 fired")

    timer2 := time.NewTimer(time.Second)
    go func() {
        <-timer2.C
        fmt.Println("Timer 2 fired")
    }()
    stop2 := timer2.Stop()
    if stop2 {
        fmt.Println("Timer 2 stopped")
    }
    // Give the `timer2` enough time to fire, if it ever
	// was going to, to show it is in fact stopped.
    time.Sleep(2 * time.Second)
}
```
Output:
```
Timer 1 fired
Timer 2 stopped
```
The <-timer1.C blocks on the timer’s channel C until it sends a value indicating that the timer fired. If a program has already received a value from t.C, the timer is known to have expired and the channel drained.

Question:
- If timer is for waiting, why can't we just use time.Sleep()?
>One reason a timer may be useful is that you can Stop the timer before it fires.You can Reset() the timer if needed. 

##### Ticker

>Timers are for when you want to do something once in the future - tickers are for when you want to do something repeatedly at regular intervals
>Tickers can be stopped like timers. Once a ticker is stopped it won’t receive any more values on its channel.

Example of a ticker that ticks periodically until we stop it:
```
package main

import (
    "fmt"
    "time"
)

func main() {

    ticker := time.NewTicker(500 * time.Millisecond)
    done := make(chan bool)

    go func() {
        for {
            select {
            case <-done:
                return
            case t := <-ticker.C:
                fmt.Println("Tick at", t)
            }
        }
    }()

    time.Sleep(1600 * time.Millisecond)
    ticker.Stop()
    done <- true
    fmt.Println("Ticker stopped")
}
```
Output:
```
Tick at 2012-09-23 11:29:56.487625 -0700 PDT
Tick at 2012-09-23 11:29:56.988063 -0700 PDT
Tick at 2012-09-23 11:29:57.488076 -0700 PDT
Ticker stopped
```
Question:
- What is the difference between Timer and Ticker?
> In case of Timer, if a value from t.C is recieved, the timer is known to have expired and the channel is drained.
> In case of Ticker, even if a value from t.C is recived, the ticker is never expired until we stop it using ticker.Stop()

Timer Example:
```
package main

import (
	"fmt"
	"time"
)

func main() {

	timer := time.NewTimer(2 * time.Second)

	for i := 0; i < 3; i++ {
		t := <-timer.C
		fmt.Println(t)
		fmt.Printf("Fired %d time(s)\n", i+1)
	}

}
```

Output:
```
2009-11-10 23:00:02 +0000 UTC m=+2.000000001
Fired 1 time(s)
fatal error: all goroutines are asleep - deadlock!
```

Ticker Example:

```
package main

import (
	"fmt"
	"time"
)

func main() {

	ticker := time.NewTicker(2 * time.Second)

	for i := 0; i < 3; i++ {
		t := <-ticker.C
		fmt.Println(t)
		fmt.Printf("Fired %d time(s)\n", i+1)
	}

}
```
Output:
```
2009-11-10 23:00:02 +0000 UTC m=+2.000000001
Fired 1 time(s)
2009-11-10 23:00:04 +0000 UTC m=+4.000000001
Fired 2 time(s)
2009-11-10 23:00:06 +0000 UTC m=+6.000000001
Fired 3 time(s)
```
