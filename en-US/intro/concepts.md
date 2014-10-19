---
root: false
name: Core Concepts
sort: 1
---

# Core Concepts

## Classic Macaron

To get up and running quickly, [`macaron.Classic`](https://gowalker.org/github.com/Unknwon/macaron#Classic) provides some reasonable defaults that work well for most of web applications:

```go
 m := macaron.Classic()
 // ... middleware and routing goes here
 m.Run()
```

Below is some of the functionality [`macaron.Classic`](https://gowalker.org/github.com/Unknwon/macaron#Classic) pulls in automatically:

- Request/response logging - [`macaron.Logger`](../middlewares/core#routing-logger)
- Panic recovery - [`macaron.Recovery`](../middlewares/core#panic-recovery)
- Static file serving - [`macaron.Static`](../middlewares/core#static-files)

## Instances

Any object with type [`macaron.Macaron`](https://gowalker.org/github.com/Unknwon/macaron#Macaron) can be seen as an instance of Macaron, you can have as many as instances you want in a single application.

## Handlers

Handlers are the heart and soul of Macaron. A handler is basically any kind of callable function:

```go
m.Get("/", func() {
	println("hello world")
})
```

### Return Values

If a handler returns something, Macaron will write the result to the current [http.ResponseWriter](http://gowalker.org/net/http#ResponseWriter) as a string:

```go
m.Get("/", func() string {
	return "hello world" // HTTP 200 : "hello world"
})
```

You can also optionally return a status code:

```go
m.Get("/", func() (int, string) {
	return 418, "i'm a teapot" // HTTP 418 : "i'm a teapot"
})
```

### Service Injection

Handlers are invoked via reflection. Macaron makes use of [Dependency Injection](http://en.wikipedia.org/wiki/Dependency_injection) to resolve dependencies in a Handlers argument list. **This makes Macaron completely  compatible with golang's [`http.HandlerFunc`](https://gowalker.org/net/http#HandlerFunc) interface.**

If you add an argument to your handler, Macaron will search its list of services and attempt to resolve the dependency via type assertion:

```go
m.Get("/", func(resp http.ResponseWriter, req *http.Request) { 
	// resp and req are injected by Macaron
	resp.WriteHeader(200) // HTTP 200
})
```

The following services are included with [macaron.Classic](https://gowalker.org/github.com/Unknwon/macaron#Classic):

- [*macaron.Context](../middlewares/core#context) - HTTP request context.
- [*log.Logger](http://gowalker.org/log#Logger) - Global logger for Macaron.
- [http.ResponseWriter](http://gowalker.org/net/http/#ResponseWriter) - HTTP Response writer interface.
- [*http.Request](http://gowalker.org/net/http/#Request) - HTTP Request.

## Macaron Env

Some Macaron handlers make use of the `macaron.Env` global variable to provide special functionality for development environments vs production environments. It is recommended that the `MACARON_ENV=production` environment variable to be set when deploying a Macaron server into a production environment.