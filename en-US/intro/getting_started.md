---
root: false
name: Getting Started
sort: 0
---

# Getting Started 

Before we get started, one thing you should know that documentation does not teach you how to use Go. Instead, explore Macaron based on the basic Go knowledge you already have.

To install Macaron:

	go get github.com/Unknwon/macaron

## Minimal Example

Create a file called `main.go`, and type following code:

```go
package main

import "github.com/Unknwon/macaron"

func main() {
	m := macaron.Classic()
	m.Get("/", func() string {
		return "Hello world!"
	})
	m.Run()
}
```

Function [`macaron.Classic`](https://gowalker.org/github.com/Unknwon/macaron#Classic) creates and returns a [Classic Macaron](concepts#classic-macaron).

Method [`m.Get`](https://gowalker.org/github.com/Unknwon/macaron#Router_Get) is for registering routes for HTTP GET method. In this case, we allow GET requests to root path `/` and has a [Handler](concepts#handlers) function to simply returns string `Hello world!` as response.

You may have questions about why the handler function can return a string as response? The magic is the [Return Values](concepts#return-values), this is a special case/syntax for responding requests by string.

Finally, we call method [`m.Run`](https://gowalker.org/github.com/Unknwon/macaron#Macaron_Run) to get server running. In default, Macaron [Instances](concepts#instances) will listen on `0.0.0.0:4000`.

Then, execute command `go run main.go`, you should see a log message is printed to the console:

	[Macaron] listening on 0.0.0.0:4000 (development)

Now, open your browser and visit [localhost:4000](http://localhost:4000), maigc happens!

## Extended Example

Let’s modify the `main.go` and do some extended exercises.

```go
package main

import (
	"log"
	"net/http"

	"github.com/Unknwon/macaron"
)

func main() {
	m := macaron.Classic()
	m.Get("/", myHandler)

	log.Println("Server is running...")
	log.Println(http.ListenAndServe("0.0.0.0:4000", m))
}

func myHandler(ctx *macaron.Context) string {
	return "the request path is: " + ctx.Req.RequestURI
}
```

If you execute command `go run main.go` again, you’ll see string `the request path is: /` is on your screen.

So what’s different now? 

First of all, we still use [Classic Macaron](concepts#classic-macaron) and register route for HTTP GET method of root path `/`. We don’t use anonymous function anymore, but a named function called `myHandler`. Notice that there is no parentheses after function name when we register route because we do not call it at that point.

The function `myHandler` accepts one argument with type [`*macaron.Context`](../middlewares/core#context) and returns a string. You may notice that we didn’t tell Macaron what arguments should pass to `myHandler` when we register routes, and if you look at the [`m.Get`](https://gowalker.org/github.com/Unknwon/macaron#Router_Get) method, you will see Macaron sees all handlers([`macaron.Handler`](https://gowalker.org/github.com/Unknwon/macaron#Handler)) as type `interface{}`. So how does Macaron know?

This is related to the concept of [Service Injection](concepts#service-injection), [`*macaron.Context`](../middlewares/core#context) is one of the default injected services, so you can use it directly. Don’t worry about how to inject your own services, it’s just not the time to tell you yet.

Like the previous example, we need to make server listen on a address. This time, we use function from Go standard library called [`http.ListenAndServe`](https://gowalker.org/net/http#ListenAndServe), which shows any Macaron [Instance](concepts#instances) is fully compatible with Go standard library.

## Go Further

You now know about how to write simple code based on Macaron, please try to modify two examples above and make sure you fully understand the words you read.

When you feel comfortable, get up and keep reading on following chapters.