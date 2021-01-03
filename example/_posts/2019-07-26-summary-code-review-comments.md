
# Code review comments: Highlights

https://github.com/golang/go/wiki/CodeReviewComments

## Code organization 

### Comment Sentences
Comments documenting declarations should be full sentences, even if that seems a little redundant. This approach makes them format well when extracted into godoc documentation. Comments should begin with the name of the thing being described and end in a period:
```
// Request represents a request to run a command.
type Request struct { ...

// Encode writes the JSON encoding of req to w.
func Encode(w io.Writer, req *Request) { ...
```

## Contexts
Values of the context.Context type carry security credentials, tracing information, deadlines, and cancellation signals across API and process boundaries. Go programs pass Contexts explicitly along the entire function call chain from incoming RPCs and HTTP requests to outgoing requests.

Most functions that use a Context should accept it as their first parameter:
```
func F(ctx context.Context, /* other arguments */) {}
```
- Don't add a Context member to a struct type; instead add a ctx parameter to each method on that type that needs to pass it along. The one exception is for methods whose signature must match an interface in the standard library or in a third party library.
- Don't create custom Context types or use interfaces other than Context in function signatures.
- Contexts are immutable, so it's fine to pass the same ctx to multiple calls that share the same deadline, cancellation signal, credentials, parent trace, etc.

## Copying
To avoid unexpected aliasing, be careful when copying a struct from another package. For example, the bytes.Buffer type contains a []byte slice. If you copy a Buffer, the slice in the copy may alias the array in the original, causing subsequent method calls to have surprising effects.

In general, do not copy a value of type T if its methods are associated with the pointer type, *T.

## Declaring Empty Slices

When declaring an empty slice, prefer
```
var t []string
```
over
```
t := []string{}
```
The former declares a nil slice value, while the latter is non-nil but zero-length. They are functionally equivalent—their len and cap are both zero—but the nil slice is the preferred style.

Note that there are limited circumstances where a non-nil but zero-length slice is preferred, such as when encoding JSON objects (a nil slice encodes to null, while []string{} encodes to the JSON array []).

When designing interfaces, avoid making a distinction between a nil slice and a non-nil, zero-length slice, as this can lead to subtle programming errors.

## Don't Panic
Don't use panic for normal error handling. Use error and multiple return values.

## Error Strings
Error strings should not be capitalized (unless beginning with proper nouns or acronyms) or end with punctuation, since they are usually printed following other context. That is, use fmt.Errorf("something bad") not fmt.Errorf("Something bad"), so that log.Printf("Reading %s: %v", filename, err) formats without a spurious capital letter mid-message.

## Goroutine Lifetimes
When you spawn goroutines, make it clear when - or whether - they exit.

Goroutines can leak by blocking on channel sends or receives: the garbage collector will not terminate a goroutine even if the channels it is blocked on are unreachable.

Even when goroutines do not leak, leaving them in-flight when they are no longer needed can cause other subtle and hard-to-diagnose problems. Sends on closed channels panic. And leaving goroutines in-flight for arbitrarily long can lead to unpredictable memory usage.
Try to keep concurrent code simple enough that goroutine lifetimes are obvious. If that just isn't feasible, document when and why the goroutines exit.

## Handle Errors
Do not discard errors using _ variables. If a function returns an error, check it to make sure the function succeeded. Handle the error, return it, or, in truly exceptional situations, panic.

## Import Blank
Packages that are imported only for their side effects (using the syntax import _ "pkg") should only be imported in the main package of a program, or in tests that require them.

## Import Dot
The import . form can be useful in tests that, due to circular dependencies, cannot be made part of the package being tested:
```
package foo_test

import (
	"bar/testutil" // also imports "foo"
	. "foo"
)
```
In this case, the test file cannot be in package foo because it uses bar/testutil, which imports foo. So we use the 'import .' form to let the file pretend to be part of package foo even though it is not. Except for this one case, do not use import . in your programs. It makes the programs much harder to read because it is unclear whether a name like Quux is a top-level identifier in the current package or in an imported package.

## Initialisms

Words in names that are initialisms or acronyms (e.g. "URL" or "NATO") have a consistent case. For example, "URL" should appear as "URL" or "url" (as in "urlPony", or "URLPony"), never as "Url". As an example: ServeHTTP not ServeHttp. For identifiers with multiple initialized "words", use for example "xmlHTTPRequest" or "XMLHTTPRequest".

This rule also applies to "ID" when it is short for "identifier" (which is pretty much all cases when it's not the "id" as in "ego", "superego"), so write "appID" instead of "appId".

## Interfaces
Go interfaces generally belong in the package that uses values of the interface type, not the package that implements those values. The implementing package should return concrete (usually pointer or struct) types: that way, new methods can be added to implementations without requiring extensive refactoring.

Do not define interfaces on the implementor side of an API "for mocking"; instead, design the API so that it can be tested using the public API of the real implementation.

Do not define interfaces before they are used: without a realistic example of usage, it is too difficult to see whether an interface is even necessary, let alone what methods it ought to contain.

// DO NOT DO IT!!!
```
package producer

type Thinger interface { Thing() bool }

type defaultThinger struct{ … }
func (t defaultThinger) Thing() bool { … }

func NewThinger() Thinger { return defaultThinger{ … } }
```
Instead return a concrete type and let the consumer mock the producer implementation.
```
package producer

type Thinger struct{ … }
func (t Thinger) Thing() bool { … }

func NewThinger() Thinger { return Thinger{ … } }
```

## Named Result Parameters
Consider what it will look like in godoc. Named result parameters like:
```
func (n *Node) Parent1() (node *Node) {}
func (n *Node) Parent2() (node *Node, err error) {}
```
will stutter in godoc; better to use:
```
func (n *Node) Parent1() *Node {}
func (n *Node) Parent2() (*Node, error) {}
```
On the other hand, if a function returns two or three parameters of the same type, or if the meaning of a result isn't clear from context, adding names may be useful in some contexts. Don't name result parameters just to avoid declaring a var inside the function; that trades off a minor implementation brevity at the cost of unnecessary API verbosity.

## Package Names
Instead, name the type File, which clients will write as chubby.File. Avoid meaningless package names like util, common, misc, api, types, and interfaces.

## Pass Values

Don't pass pointers as function arguments just to save a few bytes. If a function refers to its argument x only as *x throughout, then the argument shouldn't be a pointer. Common instances of this include passing a pointer to a string (*string) or a pointer to an interface value (*io.Reader). In both cases the value itself is a fixed size and can be passed directly. This advice does not apply to large structs, or even small structs that might grow.

## Receiver Names
The name of a method's receiver should be a reflection of its identity; often a one or two letter abbreviation of its type suffices (such as "c" or "cl" for "Client"). Don't use generic names such as "me", "this" or "self", identifiers typical of object-oriented languages that gives the method a special meaning. Be consistent, too: if you call the receiver "c" in one method, don't call it "cl" in another.

## Receiver Type

Choosing whether to use a value or pointer receiver on methods can be difficult, especially to new Go programmers. If in doubt, use a pointer, but there are times when a value receiver makes sense, usually for reasons of efficiency, such as for small unchanging structs or values of basic type. Some useful guidelines:

- If the receiver is a map, func or chan, don't use a pointer to them. If the receiver is a slice and the method doesn't reslice or reallocate the slice, don't use a pointer to it.
- If the method needs to mutate the receiver, the receiver must be a pointer.
- If the receiver is a struct that contains a sync.Mutex or similar synchronizing field, the receiver must be a pointer to avoid copying.
- If the receiver is a large struct or array, a pointer receiver is more efficient. How large is large? Assume it's equivalent to passing all its elements as arguments to the method. If that feels too large, it's also too large for the receiver.
- Can function or methods, either concurrently or when called from this method, be mutating the receiver? A value type creates a copy of the receiver when the method is invoked, so outside updates will not be applied to this receiver. If changes must be visible in the original receiver, the receiver must be a pointer.
- If the receiver is a struct, array or slice and any of its elements is a pointer to something that might be mutating, prefer a pointer receiver, as it will make the intention more clear to the reader.
- If the receiver is a small array or struct that is naturally a value type (for instance, something like the time.Time type), with no mutable fields and no pointers, or is just a simple basic type such as int or string, a value receiver makes sense. A value receiver can reduce the amount of garbage that can be generated; if a value is passed to a value method, an on-stack copy can be used instead of allocating on the heap. (The compiler tries to be smart about avoiding this allocation, but it can't always succeed.) Don't choose a value receiver type for this reason without profiling first.
- Finally, when in doubt, use a pointer receiver.

## Variable Names
Variable names in Go should be short rather than long. This is especially true for local variables with limited scope. Prefer c to lineCount. Prefer i to sliceIndex.

The basic rule: the further from its declaration that a name is used, the more descriptive the name must be. For a method receiver, one or two letters is sufficient. Common variables such as loop indices and readers can be a single letter (i, r). More unusual things and global variables need more descriptive names.

Add context to errors
Don't:
```
file, err := os.Open("foo.txt")
if err != nil {
	return err
}
```
Using the approach above can lead to unclear error messages because of missing context.

Do:
```
file, err := os.Open("foo.txt")
if err != nil {
	return fmt.Errorf("open foo.txt failed: %w", err)
}
```
## Structured logging
Don't:
```
log.Printf("Listening on :%d", port)
http.ListenAndServe(fmt.Sprintf(":%d", port), nil)
// 2017/07/29 13:05:50 Listening on :80
```
Do:
```
import "github.com/sirupsen/logrus"
// ...

logger.WithField("port", port).Info("Server is listening")
http.ListenAndServe(fmt.Sprintf(":%d", port), nil)
// {"level":"info","msg":"Server is listening","port":"7000","time":"2017-12-24T13:25:31+01:00"}
```
This is a harmless example, but using structured logging makes debugging and log parsing easier.

## Avoid global variables
Don't:
```
var db *sql.DB

func main() {
	db = // ...
	http.HandleFunc("/drop", DropHandler)
	// ...
}

func DropHandler(w http.ResponseWriter, r *http.Request) {
	db.Exec("DROP DATABASE prod")
}
```
Global variables make testing and readability hard and every method has access to them (even those, that don't need it).

Do:
```
func main() {
	db := // ...
	handlers := Handlers{DB: db}
	http.HandleFunc("/drop", handlers.DropHandler)
	// ...
}

type Handlers struct {
	DB *sql.DB
}

func (h *Handlers) DropHandler(w http.ResponseWriter, r *http.Request) {
	h.DB.Exec("DROP DATABASE prod")
}
```
Use structs to encapsulate the variables and make them available only to those functions that actually need them by making them methods implemented for that struct.

Alternatively higher-order functions can be used to inject dependencies via closures.
```
func main() {
	db := // ...
	http.HandleFunc("/drop", DropHandler(db))
	// ...
}

func DropHandler(db *sql.DB) http.HandleFunc {
	return func (w http.ResponseWriter, r *http.Request) {
		db.Exec("DROP DATABASE prod")
	}
}
```
If you really need global variables or constants, e.g., for defining errors or string constants, put them at the top of your file.

## Testing
Use an assert libary
Don't:
```
func TestAdd(t *testing.T) {
	actual := 2 + 2
	expected := 4
	if (actual != expected) {
		t.Errorf("Expected %d, but got %d", expected, actual)
	}
}
```
Do:
```
import "github.com/stretchr/testify/assert"

func TestAdd(t *testing.T) {
	actual := 2 + 2
	expected := 4
	assert.Equal(t, expected, actual)
}
```
Using assert libraries makes your tests more readable, requires less code and provides consistent error output.

## Use linters
Use all the linters included in golangci-lint to lint your projects before committing.

###
Installation - replace vX.X.X with the version you want to use
GO111MODULE=on go get github.com/golangci/golangci-lint/cmd/golangci-lint@vX.X.X
### traditional way without go module
go get -u github.com/golangci/golangci-lint/cmd/golangci-lint
### Usage in the project workspace
golangci-lint run
For detailed usage and the ci-pipeline installation guide visit golangci-lint(https://github.com/golangci/golangci-lint)

goimports
Only commit gofmt'd files. Use goimports for this to format/update the import statements as well.

## Don't over-interface
Don't:
```
type Server interface {
	Serve() error
	Some() int
	Fields() float64
	That() string
	Are([]byte) error
	Not() []string
	Necessary() error
}
```
Favour small interfaces and only expect the interfaces you need in your funcs.

Handle signals
Don't:
```
func main() {
	for {
		time.Sleep(1 * time.Second)
		ioutil.WriteFile("foo", []byte("bar"), 0644)
	}
}
```
Do:
```
func main() {
	logger := // ...
	sc := make(chan os.Signal, 1)
	done := make(chan bool)

	go func() {
		for {
			select {
			case s := <-sc:
				logger.Info("Received signal, stopping application",
					zap.String("signal", s.String()))
				done <- true
				return
			default:
				time.Sleep(1 * time.Second)
				ioutil.WriteFile("foo", []byte("bar"), 0644)
			}
		}
	}()

	signal.Notify(sc, os.Interrupt, os.Kill)
	<-done // Wait for go-routine
}
```
Handling signals allows us to gracefully stop our server, close open files and connections and therefore prevent file corruption among other things.

## Divide imports
Don't:
```
import (
	"encoding/json"
	"github.com/some/external/pkg"
	"fmt"
	"github.com/this-project/pkg/some-lib"
	"os"
)
```
Do:
```
import (
	"encoding/json"
	"fmt"
	"os"

	"github.com/bahlo/this-project/pkg/some-lib"

	"github.com/bahlo/another-project/pkg/some-lib"
	"github.com/bahlo/yet-another-project/pkg/some-lib"

	"github.com/some/external/pkg"
	"github.com/some-other/external/pkg"
)
```
Divide imports into four groups sorted from internal to external for readability:
- Standard library
- Project internal packages
- Company internal packages
- External packages

Avoid unadorned return
Don't:
```

func run() (n int, err error) {
	// ...
	return
}
```
Do:
```
func run() (n int, err error) {
	// ...
	return n, err
}
```
Named returns are good for documentation, unadorned returns are bad for readability and error-prone.
## Avoid empty interface
Don't:
```
func run(foo interface{}) {
	// ...
}
```
Empty interfaces make code more complex and unclear, avoid them where you can.

## Main first
Don't:
```
package main // import "github.com/me/my-project"

func someHelper() int {
	// ...
}

func someOtherHelper() string {
	// ...
}

func Handler(w http.ResponseWriter, r *http.Reqeust) {
	// ...
}

func main() {
	// ...
}
```
Do:
```
package main // import "github.com/me/my-project"

func main() {
	// ...
}

func Handler(w http.ResponseWriter, r *http.Reqeust) {
	// ...
}

func someHelper() int {
	// ...
}

func someOtherHelper() string {
	// ...
}
```

Putting main() first makes reading the file a lot more easier. Only the init() function should be above it.

## Use internal packages
If you're creating a cmd, consider moving libraries to internal/ to prevent import of unstable, changing packages.

- Avoid helper/util
- Use clear names and try to avoid creating a helper.go, utils.go or even package.

## Use io.WriteString
A number of important types that satisfy io.Writer also have a WriteString method, including *bytes.Buffer, *os.File and *bufio.Writer. WriteString is behavioral contract with implicit assent that passed string will be written in efficient way, without a temporary allocation. Therefore using io.WriteString may improve performance at most, and at least string will be written in any way.

Don't:
```
var w io.Writer = new(bytes.Buffer)
str := "some string"
w.Write([]byte(str))
```
Do:
```
var w io.Writer = new(bytes.Buffer)
str := "some string"
io.WriteString(w, str)
```
## Structs

Use named structs
If a struct has more than one field include field names when instantiating it.

Don't:
```
params := myStruct{
	1, 
	true,
}
```
Do:
```
params := myStruct{
	Foo: 1,
	Bar: true,
}
```
## Avoid new keyword
Using the normal syntax instead of the new keyword makes it more clear what is happening: a new instance of the struct is created MyStruct{} and we get the pointer for it with &.

Don't:
```
s := new(MyStruct)
```
Do:
```
s := &MyStruct{}
```
