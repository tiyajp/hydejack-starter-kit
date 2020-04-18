---
layout: post
title: HTTP Server
description: >
  An Introduction to HTTP Server.
image: /assets/img/http.png
noindex: true
---

![](/assets/img/server.png)


### HTTP Server

Before going to http Server, Let's understand what a server is.

Web Server
 > "Web server" can refer to hardware or software, or both of them working together.
 
1) On the hardware side, a web server is a computer that stores web server software and a website's component files (e.g. HTML documents, images, CSS stylesheets, and JavaScript files). It is connected to the Internet and supports physical data interchange with other devices connected to the web.

2) On the software side, a web server includes several parts that control how web users access hosted files, at minimum an HTTP server. An HTTP server is a piece of software that understands URLs (web addresses) and HTTP (the protocol your browser uses to view webpages). It can be accessed through the domain names (like mozilla.org) of websites it stores, and delivers their content to the end-user's device. This intercommunication is done using Hypertext Transfer Protocol (HTTP).

> At the most basic level, whenever a browser needs a file which is hosted on a web server, the browser requests the file via HTTP. When the request reaches the correct web server (hardware), the HTTP server (software) accepts request, finds the requested document (if it doesn't then a 404 response is returned), and sends it back to the browser, also through HTTP.

### HTTP Server

Writing a basic HTTP server is easy using the net/http package.
A fundamental concept in net/http servers is handlers. A handler is an object implementing the http.Handler interface. 

ListenAndServe starts an HTTP server with a given address and handler. The handler is usually nil, which means to use DefaultServeMux. Handle and HandleFunc add handlers to DefaultServeMux:

```
http.Handle("/foo", fooHandler)

http.HandleFunc("/bar", func(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
})

log.Fatal(http.ListenAndServe(":8080", nil))
```    

A basic understanding of the following concepts are neccessary for Web Server Creation
* type Handler 
* type ServeMux
    * func NewServeMux() *ServeMux
    * func (mux *ServeMux) Handle(pattern string, handler Handler)
    * func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request))
    * func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request)
* type Server
    * func (srv *Server) ListenAndServe() error
* func ListenAndServe(addr string, handler Handler) error
* type HandlerFunc
   * func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request)
   
### type Handler

A Handler responds to an HTTP request.

```
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

Handlers are responsible for writing response headers and bodies. Almost any object can be a handler, so long as it satisfies the http.Handler interface.

### type ServeMux

ServeMux is an HTTP request multiplexer.A mux is used for routing. It matches the URL of each incoming request against a list of registered patterns and calls the handler for the pattern that most closely matches the URL.
ServeMux also takes care of sanitizing the URL request path and the Host header, stripping the port number and redirecting any request containing . or .. elements or repeated slashes to an equivalent, cleaner URL.

### func NewServeMux

NewServeMux allocates and returns a new ServeMux.

```
func NewServeMux() *ServeMux
```

### func (*ServeMux) Handle

Handle registers the handler for the given pattern in the DefaultServeMux. If a handler already exists for pattern, Handle panics.
We can simply use a struct to provide the implementation. 

```
func (mux *ServeMux) Handle(pattern string, handler Handler)
```


For example:

```
func (h home) ServeHTTP(rw http.ResponseWriter, r *http.Request) {
	rw.Write([]byte("Welcome to home"))
}
```

Then attach this to the multiplexer as follows:

```
mux := http.NewServeMux()
mux.Handle("/", home{})
```

### func (*ServeMux) HandleFunc

HandleFunc registers the handler function for the given pattern in the DefaultServeMux.Handler func has two parameters. One to handle the response (http.ResponseWriter ) and another (http.Request ) to get any data or value from the request. With this second service we can read GET parameters with r.URL.Query().Get("keyword") or POST parameters in request body with r.Body.

```
func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request))
```

```
mux := http.NewServeMux()
mux.HandleFunc("/hello", func(rw http.ResponseWriter, req *http.Request) {
	rw.Write([]byte("Hello"))
})
```

### func (*ServeMux) ServeHTTP

ServeHTTP dispatches the request to the handler whose pattern most closely matches the request URL.

```
func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request)
```

### type Server

A Server defines parameters for running an HTTP server. The zero value for Server is a valid configuration.

```
type Server struct {
    // Addr optionally specifies the TCP address for the server to listen on,
    // in the form "host:port". If empty, ":http" (port 80) is used.
    Addr string
    Handler Handler // handler to invoke, http.DefaultServeMux if nil
    ...
}
```

### func (srv *Server) ListenAndServe() error

ListenAndServe listens on the TCP network address srv.Addr and then calls Serve to handle requests on incoming connections. Accepted connections are configured to enable TCP keep-alives.

If srv.Addr is blank, ":http" is used.
ListenAndServe always returns a non-nil error. After Shutdown or Close, the returned error is ErrServerClosed.

### func ListenAndServe

ListenAndServe listens on the TCP network address addr and then calls Serve with handler to handle requests on incoming connections. Accepted connections are configured to enable TCP keep-alives.
The handler is typically nil, in which case the DefaultServeMux is used.
ListenAndServe always returns a non-nil error.
```
func ListenAndServe(addr string, handler Handler) error
```


Example: HTTP Server

### Registering handlers


```
package main

import (
	"fmt"
	"net/http"
)

type welcome struct{}

func (wc welcome) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Welcome to HTTP server Mux Handle"))
}

func hello(w http.ResponseWriter, req *http.Request) {
	w.Write([]byte("Welcome to HTTP server Mux HandleFunc\nHello, How you doing?"))
}

func main() {
	mux := http.NewServeMux()
	mux.Handle("/", welcome{})
	mux.HandleFunc("/hello", hello)
	fmt.Println("Starting the server on :8080")
	http.ListenAndServe(":8080", mux)
	// http.ListenAndServe(":8080", mux) similar to 
	// server := http.Server{Addr: ":8080", Handler: mux}
	// server.ListenAndServe()
}
```

Output:
It will print 

```
Starting the server on :8080
``` 
on terminal. And on browser http://localhost:8080/ will give 
```
Welcome to HTTP server Mux Handle
``` 
and http://localhost:8080/hello will give 
``` 
Welcome to HTTP server Mux HandleFunc
Hello, How you doing?
```

Question:
What does mux.Handle() actually do? Is it responsible for calling Handler's ServeHTTP?
> No, It will not call Handler's ServeHTTP.Handle registers the handler for the given pattern.If a handler already exists for pattern, Handle panics.
See the code snippet from source code:

```
type ServeMux struct {
	mu    sync.RWMutex
	m     map[string]muxEntry
	es    []muxEntry // slice of entries sorted from longest to shortest.
	hosts bool       // whether any patterns contain hostnames
}

type muxEntry struct {
	h       Handler
	pattern string
}
// Handle registers the handler for the given pattern.
// If a handler already exists for pattern, Handle panics.
func (mux *ServeMux) Handle(pattern string, handler Handler) {
	mux.mu.Lock()
	defer mux.mu.Unlock()

	if pattern == "" {
		panic("http: invalid pattern")
	}
	if handler == nil {
		panic("http: nil handler")
	}
	if _, exist := mux.m[pattern]; exist {
		panic("http: multiple registrations for " + pattern)
	}

	if mux.m == nil {
		mux.m = make(map[string]muxEntry)
	}
	e := muxEntry{h: handler, pattern: pattern} // Register the handler for the pattern
	mux.m[pattern] = e
	if pattern[len(pattern)-1] == '/' {
		mux.es = appendSorted(mux.es, e)
	}

	if pattern[0] != '/' {
		mux.hosts = true
	}
}

func appendSorted(es []muxEntry, e muxEntry) []muxEntry {
	n := len(es)
	i := sort.Search(n, func(i int) bool {
		return len(es[i].pattern) < len(e.pattern)
	})
	if i == n {
		return append(es, e)
	}
	// we now know that i points at where we want to insert
	es = append(es, muxEntry{}) // try to grow the slice in place, any entry works.
	copy(es[i+1:], es[i:])      // Move shorter entries down
	es[i] = e
	return es
}
```

Question:
What does mux.HandleFunc() actually do?
> HandleFunc registers the handler function for the given pattern.
See the code snippet from source code:

```
// HandleFunc registers the handler function for the given pattern.
func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	if handler == nil {
		panic("http: nil handler")
	}
	mux.Handle(pattern, HandlerFunc(handler))
}

// The HandlerFunc type is an adapter to allow the use of
// ordinary functions as HTTP handlers.  If f is a function
// with the appropriate signature, HandlerFunc(f) is a
// Handler object that calls f.
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(w, req).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, req *Request) {
    f(w, req)
}
```

The call to the http.HandleFunc() function "converts" the provided function to the HandleFunc() type which has a ServeHTTP method and thus implements Handler interface and then calls the (mux *ServeMux) Handle() function similar to what happens when we call the Handle() function.Thereby registering the handler against given pattern.

Question:
Mux is passed to ListenAndServe(). How does the mux implements Hander interface?
> http.NewServeMux() returns a pointer to a serve mux (*ServeMux) and this type *ServeMux has a ServeHttp method.Therefore, any value of type *ServeMux will essentially implement the Handler interface Since ListenAndServe wants a handler of type Handler and any value of type *ServeMux implements the Handler interface, we can pass *ServeMux to the ListenAndServe function

### Default multiplexer
To make things simpler, there’s a ready-to-use multiplexer DefaultServeMux. You don't need to use an explicit ServeMux. The Handle and HandleFunc methods available in a ServeMux are also exposed as global functions in the net/http package for this purpose — you can use them the same way!
The DefaultServeMux is just a plain ol' ServeMux like we've already been using, which gets instantiated by default when the HTTP package is used.

Here's the relevant line from the Go source:

```
// ServeMux also takes care of sanitizing the URL request path,
// redirecting any request containing . or .. elements or repeated slashes
// to an equivalent, cleaner URL.
type ServeMux struct {
     mu    sync.RWMutex
     m     map[string]muxEntry
     hosts bool // whether any patterns contain hostnames
}

type muxEntry struct {
     explicit bool
     h        Handler
     pattern  string
}

// NewServeMux allocates and returns a new ServeMux.
func NewServeMux() *ServeMux { return new(ServeMux) }

// DefaultServeMux is the default ServeMux used by Serve.
var DefaultServeMux = &defaultServeMux

var defaultServeMux ServeMux
```

Example:

```
package main

import (
	"fmt"
	"net/http"
)

type welcome struct{}

func (wc welcome) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Welcome to HTTP server Mux Handle"))
}

func hello(w http.ResponseWriter, req *http.Request) {
	w.Write([]byte("Welcome to HTTP server Mux HandleFunc\nHello, How you doing?"))
}

func main() {

	http.Handle("/", welcome{})
	http.HandleFunc("/hello", hello)
	fmt.Println("Starting the server on :8080")
	http.ListenAndServe(":8080", nil)
}
```

 Output of the above code is similar to the HTTP Server above.
 Generally you shouldn't use the DefaultServeMux because it poses a security risk.

Because the DefaultServeMux is stored in a global variable, any package is able to access it and register a route including any third-party packages that your application imports. If one of those third-party packages is compromised, they could use the DefaultServeMux to expose a malicious handler to the web.

Question:
How does DefaultServeMux get set when the second argument to ListenAndServe() is nil?

The following code snippet has the answer:

```
// serverHandler delegates to either the server's Handler or
// DefaultServeMux and also handles "OPTIONS *" requests.
type serverHandler struct {
	srv *Server
}
func (sh serverHandler) ServeHTTP(rw ResponseWriter, req *Request) {
	handler := sh.srv.Handler
	if handler == nil {
		handler = DefaultServeMux
	}
	if req.RequestURI == "*" && req.Method == "OPTIONS" {
		handler = globalOptionsHandler{}
	}
	handler.ServeHTTP(rw, req)
}
```

### type HandlerFunc
The HandlerFunc type is an adapter to allow the use of ordinary functions as HTTP handlers. If f is a function with the appropriate signature, HandlerFunc(f) is a Handler that calls f.ie. If you want to use a standalone function without declaring a struct,http.Handlerfunc comes to the rescue.

```
type HandlerFunc func(ResponseWriter, *Request)
```

### func (HandlerFunc) ServeHTTP

```
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(w, req).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, req *Request) {
    f(w, req)
}
```

ServeHTTP calls f(w, r).

Example:

```
package main

import (
	"fmt"
	"net/http"
)

func hello(w http.ResponseWriter, req *http.Request) {
	w.Write([]byte("Hello, How you doing?"))
}

func main() {

	http.Handle("/", http.HandlerFunc(hello))
	http.HandleFunc("/hello", hello)
	fmt.Println("Starting the server on :8080")
	http.ListenAndServe(":8080", nil)
}
```

Output:
Both http://localhost:8080/ and http://localhost:8080/hello will print "Hello, How you doing?"

