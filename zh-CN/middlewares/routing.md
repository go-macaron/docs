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
- ...但是，匹配范围较小的路由优先级比匹配范围大的优先级高（详见 **匹配优先级**）。
- 最先被定义的路由将会首先被用户请求匹配并调用。

在一些时候，每当 GET 方法被注册的时候，都会需要注册一个一模一样的 HEAD 方法。为了达到减少代码的目的，您可以使用一个名为 [`SetAutoHead`](https://gowalker.org/github.com/Unknwon/macaron#Router_SetAutoHead) 的方法来协助您自动注册：

```go
m := New()
m.SetAutoHead(true)
m.Get("/", func() string {
	return "GET"
}) // 路径 "/" 的 HEAD 也已经被自动注册
```

如果您想要使用子路径但让路由代码保持简洁，可以调用 `m.SetURLPrefix(suburl)`。

## 命名参数

路由模型可能包含参数列表, 可以通过  [`*Context.Params`](https://gowalker.org/github.com/Unknwon/macaron#Context_Params) 来获取:

### 占位符

使用一个特定的名称来代表路由的某个部分：

```go
m.Get("/hello/:name", func(ctx *macaron.Context) string {
	return "Hello " + ctx.Params(":name")
})

m.Get("/date/:year/:month/:day", func(ctx *macaron.Context) string {
	return fmt.Sprintf("Date: %s/%s/%s", ctx.Params(":year"), ctx.Params(":month"), ctx.Params(":day"))
})
```

当然，想要偷懒的时候可以将 `:` 前缀去掉：

```go
m.Get("/hello/:name", func(ctx *macaron.Context) string {
	return "Hello " + ctx.Params("name")
})

m.Get("/date/:year/:month/:day", func(ctx *macaron.Context) string {
	return fmt.Sprintf("Date: %s/%s/%s", ctx.Params("year"), ctx.Params("month"), ctx.Params("day"))
})
```

### 全局匹配

路由匹配可以通过全局匹配的形式:

```go
m.Get("/hello/*", func(ctx *macaron.Context) string {
	return "Hello " + ctx.Params("*")
})
```

那么，如果将 `*` 放在路由中间会发生什么呢？

```go
m.Get("/date/*/*/*/events", func(ctx *macaron.Context) string {
	return fmt.Sprintf("Date: %s/%s/%s", ctx.Params("*0"), ctx.Params("*1"), ctx.Params("*2"))
})
```

### 正则表达式

您还可以使用正则表达式来书写路由规则：

- 常规匹配：

	```go
	m.Get("/user/:username([\\w]+)", func(ctx *macaron.Context) string {
		return fmt.Sprintf("Hello %s", ctx.Params(":username"))
	})

	m.Get("/user/:id([0-9]+)", func(ctx *macaron.Context) string {
		return fmt.Sprintf("User ID: %s", ctx.Params(":id"))
	})

	m.Get("/user/*.*", func(ctx *macaron.Context) string {
		return fmt.Sprintf("Last part is: %s, Ext: %s", ctx.Params(":path"), ctx.Params(":ext"))
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

## 匹配优先级

以下为从高到低的不同模式的匹配优先级：

- 静态路由：
	- `/`
	- `/home`
- 正则表达式路由：
	- `/(.+).html`
	- `/([0-9]+).css`
- 路径-后缀路由：
	- `/*.*`
- 占位符路由：
	- `/:id`
	- `/:name`
- 全局匹配路由：
	- `/*`

其它说明：

- 相同模式的匹配优先级是根据添加的先后顺序决定的。
- 层级相对明确的模式匹配优先级要高于相对模糊的模式：
	- `/*/*/events` > `/*`

### 构建 URL 路径

您可以通过 [`*Route.Name`](https://gowalker.org/github.com/Unknwon/macaron#Route_Name) 方法配合命名参数来构建 URL 路径，不过首先需要为路由命名：

```go
// ...
m.Get("/users/:id([0-9]+)/:name:string.profile", handler).Name("user_profile")
m.Combo("/api/:user/:repo").Get(handler).Post(handler).Name("user_repo")
// ...
```

然后通过 [`*Router.URLFor`](https://gowalker.org/github.com/Unknwon/macaron#Router_URLFor) 方法来为指定名称的路由构建 URL 路径：

```go
// ...
func handler(ctx *macaron.Context) {
	// /users/12/unknwon.profile
	userProfile := ctx.URLFor("user_profile", ":id", "12", ":name", "unknwon")
	// /api/unknwon/macaron
	userRepo := ctx.URLFor("user_repo", ":user", "unknwon", ":repo", "macaron")
}
// ...
```

#### 配合 Go 模板引擎使用

```go
// ...
m.Use(macaron.Renderer(macaron.RenderOptions{
	Funcs:      []template.FuncMap{map[string]interface{}{
		"URLFor": m.URLFor,	
	}},
}))
// ...
```

#### 配合 Pongo2 模板引擎使用

```go
// ...
ctx.Data["URLFor"] = ctx.URLFor
ctx.HTML(200, "home")
// ...
```

## 高级路由定义

路由处理器可以被相互叠加使用, 例如很有用的地方可以是在验证和授权的时候:

```go
m.Get("/secret", authorize, func() {
	// this will execute as long as authorize doesn't write a response
})
```

让我们来看一个比较极端的例子：

```go
package main

import (
	"fmt"

	"github.com/Unknwon/macaron"
)

func main() {
	m := macaron.Classic()
	m.Get("/",
		func(ctx *macaron.Context) {
			ctx.Data["Count"] = 1
		},
		func(ctx *macaron.Context) {
			ctx.Data["Count"] = ctx.Data["Count"].(int) + 1
		},
		func(ctx *macaron.Context) {
			ctx.Data["Count"] = ctx.Data["Count"].(int) + 1
		},
		func(ctx *macaron.Context) {
			ctx.Data["Count"] = ctx.Data["Count"].(int) + 1
		},
		func(ctx *macaron.Context) {
			ctx.Data["Count"] = ctx.Data["Count"].(int) + 1
		},
		func(ctx *macaron.Context) string {
			return fmt.Sprintf("There are %d handlers before this", ctx.Data["Count"])
		},
	)
	m.Run()
}
```

先意淫下结果？没错，输出结果会是 `There are 5 handlers before this`。Macaron 并没有对您可以使用多少个处理器有一个硬性的限制。不过，Macaron 又是怎么知道什么时候停止调用下一个处理器的呢？

想要回答这个问题，我们先来看下下一个例子：

```go
package main

import (
	"fmt"

	"github.com/Unknwon/macaron"
)

func main() {
	m := macaron.Classic()
	m.Get("/",
		func(ctx *macaron.Context) {
			ctx.Data["Count"] = 1
		},
		func(ctx *macaron.Context) {
			ctx.Data["Count"] = ctx.Data["Count"].(int) + 1
		},
		func(ctx *macaron.Context) {
			ctx.Data["Count"] = ctx.Data["Count"].(int) + 1
		},
		func(ctx *macaron.Context) {
			ctx.Data["Count"] = ctx.Data["Count"].(int) + 1
		},
		func(ctx *macaron.Context) string {
			return fmt.Sprintf("There are %d handlers before this", ctx.Data["Count"])
		},
		func(ctx *macaron.Context) string {
			return fmt.Sprintf("There are %d handlers before this", ctx.Data["Count"])
		},
	)
	m.Run()
}
```

在这个例子中，输出结果将会变成 `There are 4 handlers before this`，而最后一个处理器永远也不会被调用。这是为什么呢？因为我们已经在第 5 个处理器中向响应流写入了内容。所以说，一旦任一处理器向响应流写入任何内容，Macaron 将不会再调用下一个处理器。

### 组路由

路由还可以通过 [`macaron.Group`](https://gowalker.org/github.com/Unknwon/macaron#Router_Group) 来注册组路由：

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

    m.Group("/chapters", func() {
	    m.Get("/:id", GetBooks)
	    m.Post("/new", NewBook)
	    m.Put("/update/:id", UpdateBook)
	    m.Delete("/delete/:id", DeleteBook)
	}, MyMiddleware3, MyMiddleware4)
}, MyMiddleware1, MyMiddleware2)
```

同样的，Macaron 不在乎您使用多少层嵌套的组路由，或者多少个组级别处理器（中间件）。
