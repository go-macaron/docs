---
root: false
name: Custom Services
sort: 0
---

# Custom Services

Services are objects that are available to be injected into a handler's argument list. You can map a service on a **Global** or **Request** level.

#### Global Mapping

A Macaron instance implements the [`inject.Injector`](https://gowalker.org/github.com/Unknwon/macaron/inject#Injector) interface, so mapping a service is easy:

```go
db := &MyDatabase{}
m := macaron.Classic()
m.Map(db) // Service will be available to all handlers as *MyDatabase
m.Get("/", func(db *MyDatabase) {
	// Operations with db.
})
m.Run()
```

#### Request-Level Mapping

Mapping on the request level can be done in a handler via [`*macaron.Context`](https://gowalker.org/github.com/Unknwon/macaron#Context):

```go
func MyCustomLoggerHandler(ctx *macaron.Context) {
	logger := &MyCustomLogger{ctx.Req}
	ctx.Map(logger) // mapped as *MyCustomLogger
}

func main() {
	//...
	m.Get("/", MyCustomLoggerHandler, func(logger *MyCustomLogger) {
		// Operations with logger.
	})
	m.Get("/panic", func(logger *MyCustomLogger) {
		// This will panic because no logger service maps to this request.
	})
	//...
}
```

#### Mapping values to Interfaces

One of the most powerful parts about services is the ability to map a service to an interface. For instance, if you wanted to override the [`http.ResponseWriter`](http://gowalker.org/net/http#ResponseWriter) with an object that wrapped it and performed extra operations, you can write the following handler:

```go
func WrapResponseWriter(ctx *macaron.Context) {
	rw := NewSpecialResponseWriter(ctx.Resp)
	// override ResponseWriter with our wrapper ResponseWriter
	ctx.MapTo(rw, (*http.ResponseWriter)(nil)) 
}
```

In this way, your code can enjoy new custom service feature without any change. Plus, allow more custom implementations of same type of services.
