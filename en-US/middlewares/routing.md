---
root: false
name: Routing
sort: 2
---

# Routing

In Macaron, a route is an HTTP method paired with a URL-matching pattern.
Each route can take one or more handler methods:

```go
m.Get("/", func() {
	// show something
})

m.Patch("/", func() {
	// update something
})

m.Post("/", func() {
	// create something
})

m.Put("/", func() {
	// replace something
})

m.Delete("/", func() {
	// destroy something
})

m.Options("/", func() {
	// http options
})

m.Any("/", func() {
	// do anything
})

m.Route("/", "GET,POST", func() {
	// combine something
})

m.Combo("/").
	Get(func() string { return "GET" }).
	Patch(func() string { return "PATCH" }).
	Post(func() string { return "POST" }).
	Put(func() string { return "PUT" }).
	Delete(func() string { return "DELETE" }).
	Options(func() string { return "OPTIONS" }).
	Head(func() string { return "HEAD" })

m.NotFound(func() {
	// handle 404
})
```

Routes are matched in the order they are defined. The first route that
matches the request is invoked.

if you want to use suburl without having a huge group indent, use `m.SetURLPrefix(suburl)`.

Route patterns may include named parameters, accessible via the [*Context.Params](https://gowalker.org/github.com/Unknwon/macaron#Context_Params):

```go
m.Get("/hello/:name", func(ctx *macaron.Context) string {
	return "Hello " + ctx.Params(":name")
})
```

Routes can be matched with globs:

```go
m.Get("/hello/*", func(ctx *macaron.Context) string {
	return "Hello " + ctx.Params("*")
})
```

Regular expressions can be used as well:

```go
m.Get("/user/:username([\w]+)", func(ctx *macaron.Context) string {
	return fmt.Sprintf ("Hello %s", ctx.Params(":username"))
})

m.Get("/user/:username([\w]+)", func(ctx *macaron.Context) string {
	return fmt.Sprintf ("Hello %s", ctx.Params(":username"))
})

m.Get("/cms_:id([0-9]+).html", func(ctx *macaron.Context) string {
	return fmt.Sprintf ("The ID is %s", ctx.Params(":id"))
})
```

Route handlers can be stacked on top of each other, which is useful for things like authentication and authorization:

```go
m.Get("/secret", authorize, func() {
	// this will execute as long as authorize doesn't write a response
})
```

Route groups can be added too using the Group method:

```go
m.Group("/books", func() {
    m.Get("/:id", GetBooks)
    m.Post("/new", NewBook)
    m.Put("/update/:id", UpdateBook)
    m.Delete("/delete/:id", DeleteBook)
    
    m.Group("/chapters", func() {
	    m.Get("/:id", GetBooks)
	    m.Post("/new", NewBook)
	    m.Put("/update/:id", UpdateBook)
	    m.Delete("/delete/:id", DeleteBook)
	})
})
```

Just like you can pass middlewares to a handler you can pass middlewares to groups:

```go
m.Group("/books", func() {
    m.Get("/:id", GetBooks)
    m.Post("/new", NewBook)
    m.Put("/update/:id", UpdateBook)
    m.Delete("/delete/:id", DeleteBook)
}, MyMiddleware1, MyMiddleware2)
```
