---
root: false
name: 路由模块
sort: 2
---

# 路由模块

在 Macaron 中, 路由是一个 HTTP 方法配对一个 URL 匹配模型. 每一个路由可以对应一个或多个处理器方法:

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
	// 自定义 404 处理逻辑
})
```

几点说明：

- 路由匹配的顺序是按照他们被定义的顺序执行的，
- ...但是，匹配范围较小的路由优先级比匹配范围大的优先级高（例如：固定 URL > 正则 URL）。
- 最先被定义的路由将会首先被用户请求匹配并调用。

如果您想要使用子路径但让路由代码保持简洁，可以调用 `m.SetURLPrefix(suburl)`。

路由模型可能包含参数列表, 可以通过  [`*Context.Params`](https://gowalker.org/github.com/Unknwon/macaron#Context_Params) 来获取:

```go
m.Get("/hello/:name", func(ctx *macaron.Context) string {
	return "Hello " + ctx.Params(":name")
})
```

路由匹配可以通过全局匹配的形式:

```go
m.Get("/hello/*", func(ctx *macaron.Context) string {
	return "Hello " + ctx.Params("*")
})
```

您还可以使用正则表达式来书写路由规则：

- 常规匹配：

	```go
	m.Get("/user/:username([\w]+)", func(ctx *macaron.Context) string {
		return fmt.Sprintf("Hello %s", ctx.Params(":username"))
	})
	
	m.Get("/user/:id([0-9]+)", func(ctx *macaron.Context) string {
		return fmt.Sprintf("User ID: %s", ctx.Params(":id"))
	})
	
	m.Get("/user/*.*", func(ctx *macaron.Context) string {
		return fmt.Sprintf("Last part is: %s", ctx.Params(":path"), ctx.Params(":ext"))
	})
	```

- 混合匹配：

	```go
	m.Get("/cms_:id([0-9]+).html", func(ctx *macaron.Context) string {
		return fmt.Sprintf("The ID is %s", ctx.Params(":id"))
	})
	```

- 可选匹配：
	- `/user/?:id` 可同时匹配 `/user/` 和 `/user/123`。
- 简写：
	- `/user/:id:int`：`:int` 是 `([0-9]+)` 正则的简写。
	- `/user/:name:string`：`:string` 是 `([\w]+)` 正则的简写。

## 高级路由定义

路由处理器可以被相互叠加使用, 例如很有用的地方可以是在验证和授权的时候:

```go
m.Get("/secret", authorize, func() {
	// this will execute as long as authorize doesn't write a response
})
```

路由还可以通过路由组来进行注册：

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

同样的，您可以为某一组路由设置集体的中间件：

```go
m.Group("/books", func() {
    m.Get("/:id", GetBooks)
    m.Post("/new", NewBook)
    m.Put("/update/:id", UpdateBook)
    m.Delete("/delete/:id", DeleteBook)
}, MyMiddleware1, MyMiddleware2)
```
