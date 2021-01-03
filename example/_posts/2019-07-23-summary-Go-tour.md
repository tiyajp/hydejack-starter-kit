
### Go Tour: Highlights


### #1 Struct fields can be accessed through a struct pointer.

[Struct Fields](https://tour.golang.org/moretypes/4)

### #2 Arrays

The type [n]T is an array of n values of type T.

```
var a [10]int
```
declares a variable a as an array of ten integers.

An array's length is part of its type, so arrays cannot be resized. This seems limiting, but don't worry; Go provides a convenient way of working with arrays.

[Arrays](https://tour.golang.org/moretypes/6)

### #3 Slices

- An array has a fixed size. 
- A slice, on the other hand, is a dynamically-sized, flexible view into the elements of an array. In practice, slices are much more common than arrays.

> The type []T is a slice with elements of type T.

A slice is formed by specifying two indices, a low and high bound, separated by a colon:
[Slices](https://tour.golang.org/moretypes/7)


Slices are like references to arrays
> A slice does not store any data, it just describes a section of an underlying array.

Changing the elements of a slice modifies the corresponding elements of its underlying array.
Other slices that share the same underlying array will see those changes.

https://tour.golang.org/moretypes/8

### Slice literals

A slice literal is like an array literal without the length.

This is an array literal:

```
[3]bool{true, true, false}
```

And this creates the same array as above, then builds a slice that references it:
```
[]bool{true, true, false}
```

https://tour.golang.org/moretypes/9


### Slice length and capacity

- A slice has both a length and a capacity.
- The length of a slice is the number of elements it contains.
- The capacity of a slice is the number of elements in the underlying array, counting from the first element in the slice.
- The length and capacity of a slice s can be obtained using the expressions len(s) and cap(s).
- You can extend a slice's length by re-slicing it, provided it has sufficient capacity.

https://tour.golang.org/moretypes/11 

> Creating a slice with make
> Slices can be created with the built-in make function; this is how you create dynamically-sized arrays.

The make function allocates a zeroed array and returns a slice that refers to that array:

```
a := make([]int, 5)  // len(a)=5
To specify a capacity, pass a third argument to make:

b := make([]int, 0, 5) // len(b)=0, cap(b)=5

b = b[:cap(b)] // len(b)=5, cap(b)=5
b = b[1:]      // len(b)=4, cap(b)=4
```

https://tour.golang.org/moretypes/13

### Appending to a slice

It is common to append new elements to a slice, and so Go provides a built-in append function. The documentation of the built-in package describes append.

```
func append(s []T, vs ...T) []T
```

The first parameter s of append is a slice of type T, and the rest are T values to append to the slice.

The resulting value of append is a slice containing all the elements of the original slice plus the provided values.

If the backing array of s is too small to fit all the given values a bigger array will be allocated. The returned slice will point to the newly allocated array.

### #4 Range

>The range form of the for loop iterates over a slice or map.

When ranging over a slice, two values are returned for each iteration. The first is the index, and the second is a copy of the element at that index.
https://tour.golang.org/moretypes/16


### #5 Maps

>The make function returns a map of the given type, initialized and ready for use.

```
m[key] = elem
```
Retrieve an element:

```
elem = m[key]
```
Delete an element:

```
delete(m, key)
```
Test that a key is present with a two-value assignment:

```
elem, ok = m[key]
```
If key is in m, ok is true. If not, ok is false.
If key is not in the map, then elem is the zero value for the map's element type.

https://tour.golang.org/moretypes/22

###  Function values

> Functions are values too. They can be passed around just like other values.

- Function values may be used as function arguments and return values.

https://tour.golang.org/moretypes/24

### Function closures

> Go functions may be closures. A closure is a function value that references variables from outside its body. The function may access and assign to the referenced variables; in this sense the function is "bound" to the variables.

https://tour.golang.org/moretypes/25

### Methods and interfaces

### Methods

> Go does not have classes. However, you can define methods on types.
A method is a function with a special receiver argument.
Methods are functions

- You can declare a method on non-struct types, too.
- You can only declare a method with a receiver whose type is defined in the same package as the method. 
- You cannot declare a method with a receiver whose type is defined in another package (which includes the built-in types such as int).

https://tour.golang.org/methods/3

### Pointer receivers
> Methods with pointer receivers can modify the value to which the receiver points.

Since methods often need to modify their receiver, pointer receivers are more common than value receivers.
With a value receiver, the Scale method operates on a copy of the original Vertex value. (This is the same behavior as for any other function argument


### Methods and pointer indirection

- functions with a pointer argument must take a pointer:
- while methods with pointer receivers take either a value or a pointer as the receiver when they are called.
- methods with value receivers take either a value or a pointer as the receiver when they are called:

### Choosing a value or pointer receiver

There are two reasons to use a pointer receiver.

- The first is so that the method can modify the value that its receiver points to.
- The second is to avoid copying the value on each method call. This can be more efficient if the receiver is a large struct, for example.

### Interfaces
> A value of interface type can hold any value that implements those methods.
Interfaces are implemented implicitly
A type implements an interface by implementing its methods. There is no explicit declaration of intent, no "implements" keyword.

### Interface values

Under the hood, interface values can be thought of as a tuple of a value and a concrete type:

``` 
(value, type)
```
- An interface value holds a value of a specific underlying concrete type.
- Calling a method on an interface value executes the method of the same name on its underlying type.

### Interface values with nil underlying values

> If the concrete value inside the interface itself is nil, the method will be called with a nil receiver.

In some languages this would trigger a null pointer exception, but in Go it is common to write methods that gracefully handle being called with a nil receiver (as with the method M in this example.)

Note that an interface value that holds a nil concrete value is itself non-nil.
https://tour.golang.org/methods/12

### Nil interface values

> A nil interface value holds neither value nor concrete type.

- Calling a method on a nil interface is a run-time error because there is no type inside the interface tuple to indicate which concrete method to call.
https://tour.golang.org/methods/13

### The empty interface
> The interface type that specifies zero methods is known as the empty interface:
```
interface{}
```
An empty interface may hold values of any type. (Every type implements at least zero methods.)

- Empty interfaces are used by code that handles values of unknown type. For example, fmt.Print takes any number of arguments of type interface{}.

### Type assertion

> A type assertion provides access to an interface value's underlying concrete value.
```
t := i.(T)
```
This statement asserts that the interface value i holds the concrete type T and assigns the underlying T value to the variable t.

If i does not hold a T, the statement will trigger a panic.

To test whether an interface value holds a specific type, a type assertion can return two values: the underlying value and a boolean value that reports whether the assertion succeeded.
```
t, ok := i.(T)
```
### Type switches

A type switch is a construct that permits several type assertions in series.

A type switch is like a regular switch statement, but the cases in a type switch specify types (not values), and those values are compared against the type of the value held by the given interface value.
https://tour.golang.org/methods/16

### Stringers

One of the most ubiquitous interfaces is Stringer defined by the fmt package.
```
type Stringer interface {
    String() string
}
```
> A Stringer is a type that can describe itself as a string. The fmt package (and many others) look for this interface to print values.
https://tour.golang.org/methods/17

### Error

```
type error interface {
    Error() string
}
```
> The error type is a built-in interface similar to fmt.Stringer:

```
package main

import (
	"fmt"
	"time"
)

type MyError struct {
	When time.Time
	What string
}

func (e *MyError) Error() string {
	return fmt.Sprintf("at %v, %s",
		e.When, e.What)
}

func run() error {                    // return error interface
	return &MyError{
		time.Now(),
		"it didn't work",
	}
}

func main() {
	if err := run(); err != nil {
		fmt.Println(err).  // call err.Error()
	}
}
```
### Reader

> The io package specifies the io.Reader interface, which represents the read end of a stream of data.


The io.Reader interface has a Read method:
```
func (T) Read(b []byte) (n int, err error)
```
Read populates the given byte slice with data and returns the number of bytes populated and an error value. It returns an io.EOF error when the stream ends.
https://tour.golang.org/methods/21

## Goroutines

> A goroutine is a lightweight thread managed by the Go runtime.
```
go f(x, y, z)
// starts a new goroutine running
f(x, y, z)
// The evaluation of f, x, y, and z happens in the current goroutine and the execution of f happens in the new goroutine.
```

Goroutines run in the same address space, so access to shared memory must be synchronized. The sync package provides useful primitives, although you won't need them much in Go as there are other primitives

https://tour.golang.org/concurrency/1

## Channels

> Channels are a typed conduit through which you can send and receive values with the channel operator, "<-"

- Like maps and slices, channels must be created before use:
```
ch := make(chan int)
```
By default, sends and receives block until the other side is ready. This allows goroutines to synchronize without explicit locks or condition variables.
https://tour.golang.org/concurrency/2

### Buffered Channels
Channels can be buffered. Provide the buffer length as the second argument to make to initialize a buffered channel:
```
ch := make(chan int, 100)
```
> Sends to a buffered channel block only when the buffer is full. Receives block when the buffer is empty.
https://tour.golang.org/concurrency/3

### Range and Close
> A sender can close a channel to indicate that no more values will be sent.  Receivers can test whether a channel has been closed by assigning a second parameter to the receive expression: after
```
v, ok := <-ch
```
ok is false if there are no more values to receive and the channel is closed.
**Note: Only the sender should close a channel, never the receiver. Sending on a closed channel will cause a panic**

Another note: Channels aren't like files; you don't usually need to close them. Closing is only necessary when the receiver must be told there are no more values coming, such as to terminate a range loop.

```
package main

import (
	"fmt"
)

func fibonacci(n int, c chan int) {
	x, y := 0, 1
	for i := 0; i < n; i++ {
		c <- x
		x, y = y, x+y
	}
	close(c)
}

func main() {
	c := make(chan int, 10)
	go fibonacci(cap(c), c)
	for i := range c {
		fmt.Println(i)
	}
}
```
The loop for i := range c receives values from the channel repeatedly until it is closed.
Commenting close(c) will cause panic


## Select

> The select statement lets a goroutine wait on multiple communication operations.

A select blocks until one of its cases can run, then it executes that case. It chooses one at random if multiple are ready.
```
package main

import "fmt"

func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
		}
	}
}

func main() {
	c := make(chan int)
	quit := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 0
	}()
	fibonacci(c, quit)
}
```

### Default Selection

> The default case in a select is run if no other case is ready.

**Use a default case to try a send or receive without blocking**:

select {
case i := <-c:
    // use i
default:
    // receiving from c would block
}
https://tour.golang.org/concurrency/6
