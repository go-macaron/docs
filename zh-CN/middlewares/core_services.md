---
name: 核心服务
---

# 核心服务

Macaron 会注入一些默认服务来驱动您的应用，这些服务被称之为 **核心服务**。也就是说，您可以直接使用它们作为处理器参数而不需要任何附加工作。

## 请求上下文（Context）

该服务通过类型 [`*macaron.Context`](https://gowalker.org/gopkg.in/macaron.v1#Context) 来体现。这是 Macaron 最为核心的服务，您的任何操作都是基于它之上。该服务包含了您所需要的请求对象、响应流、模板引擎接口、数据存储和注入与获取其它服务。

使用方法：

```go
package main

import "gopkg.in/macaron.v1"

func Home(ctx *macaron.Context) {
	// ...
}
```

### Next()

方法 [`Context.Next`](https://gowalker.org/gopkg.in/macaron.v1#Context_Next)  是一个可选的功能，它可以用于中间件处理器暂时放弃执行，等待其他的处理器都执行完毕后继续执行。这样就可以很好的处理在 HTTP 请求完成后需要做的操作：

```go
// log before and after a request
m.Use(func(ctx *macaron.Context, log *log.Logger){
	log.Println("before a request")

	ctx.Next()

	log.Println("after a request")
})
```

### Cookie

最基本的 Cookie 用法：

- [`*macaron.Context.SetCookie`](https://gowalker.org/gopkg.in/macaron.v1#Context_SetCookie)
- [`*macaron.Context.GetCookie`](https://gowalker.org/gopkg.in/macaron.v1#Context_GetCookie)、[`*macaron.Context.GetCookieInt`](https://gowalker.org/gopkg.in/macaron.v1#Context_GetCookieInt)、[`*macaron.Context.GetCookieInt64`](https://gowalker.org/gopkg.in/macaron.v1#Context_GetCookieInt64)、[`*macaron.Context.GetCookieFloat64`](https://gowalker.org/gopkg.in/macaron.v1#Context_GetCookieFloat64)

使用方法：

```go
// ...
m.Get("/set", func(ctx *macaron.Context) {
	ctx.SetCookie("user", "Unknwon", 1)
})

m.Get("/get", func(ctx *macaron.Context) string {
	return ctx.GetCookie("user")
})
// ...
```

使用以下顺序的参数来设置更多的属性：`SetCookie(<name>, <value>, <max age>, <path>, <domain>, <secure>, <http only>)`。

因此，设置 Cookie 最完整的用法为：`SetCookie("user", "unknwon", 999, "/", "localhost", true, true)`。

需要注意的是，参数的顺序是固定的。

如果需要更加安全的 Cookie 机制，可以先使用 [`macaron.SetDefaultCookieSecret`](https://gowalker.org/gopkg.in/macaron.v1#Macaron_SetDefaultCookieSecret) 设定密钥，然后使用：

- [`*macaron.Context.SetSecureCookie`](https://gowalker.org/gopkg.in/macaron.v1#Context_SetSecureCookie)
- [`*macaron.Context.GetSecureCookie`](https://gowalker.org/gopkg.in/macaron.v1#Context_GetSecureCookie)

这两个方法将会自动使用您设置的默认密钥进行加密/解密 Cookie 值。

使用方法：

```go
// ...
m.SetDefaultCookieSecret("macaron")
m.Get("/set", func(ctx *macaron.Context) {
	ctx.SetSecureCookie("user", "Unknwon", 1)
})

m.Get("/get", func(ctx *macaron.Context) string {
	name, _ := ctx.GetSecureCookie("user")
	return name
})
// ...
```

对于那些对安全性要求特别高的应用，可以为每次设置 Cookie 使用不同的密钥加密/解密：

- [`*macaron.Context.SetSuperSecureCookie`](https://gowalker.org/gopkg.in/macaron.v1#Context_SetSuperSecureCookie)
- [`*macaron.Context.GetSuperSecureCookie`](https://gowalker.org/gopkg.in/macaron.v1#Context_GetSuperSecureCookie)

使用方法：

```go
// ...
m.Get("/set", func(ctx *macaron.Context) {
	ctx.SetSuperSecureCookie("macaron", "user", "Unknwon", 1)
})

m.Get("/get", func(ctx *macaron.Context) string {
	name, _ := ctx.GetSuperSecureCookie("macaron", "user")
	return name
})
// ...
```

### 其它辅助方法

- 设置/获取 URL 参数：[`ctx.SetParams`](https://gowalker.org/gopkg.in/macaron.v1#Context_SetParams) / [`ctx.Params`](https://gowalker.org/gopkg.in/macaron.v1#Context_Params)、[`ctx.ParamsEscape`](https://gowalker.org/gopkg.in/macaron.v1#Context_ParamsEscape)、[`ctx.ParamsInt`](https://gowalker.org/gopkg.in/macaron.v1#Context_ParamsInt)、[`ctx.ParamsInt64`](https://gowalker.org/gopkg.in/macaron.v1#Context_ParamsInt64)、[`ctx.ParamsFloat64`](https://gowalker.org/gopkg.in/macaron.v1#Context_ParamsFloat64)
- 获取查询参数：[`ctx.Query`](https://gowalker.org/gopkg.in/macaron.v1#Context_ctx.Query)、[`ctx.QueryEscape`](https://gowalker.org/gopkg.in/macaron.v1#Context_ctx.QueryEscape)、[`ctx.QueryInt`](https://gowalker.org/gopkg.in/macaron.v1#Context_ctx.QueryInt)、[`ctx.QueryInt64`](https://gowalker.org/gopkg.in/macaron.v1#Context_ctx.QueryInt64)、[`ctx.QueryFloat64`](https://gowalker.org/gopkg.in/macaron.v1#Context_ctx.QueryFloat64)、[`ctx.QueryStrings`](https://gowalker.org/gopkg.in/macaron.v1#Context_ctx.QueryStrings)、[`ctx.QueryTrim`](https://gowalker.org/gopkg.in/macaron.v1#Context_QueryTrim)
- 服务内容或文件：[`ctx.ServeContent`](https://gowalker.org/gopkg.in/macaron.v1#Context_ServeContent)、[`ctx.ServeFile`](https://gowalker.org/gopkg.in/macaron.v1#Context_ServeFile)、[`ctx.ServeFile`](https://gowalker.org/gopkg.in/macaron.v1#Context_ServeFile), [`ctx.ServeFileContent`](https://gowalker.org/gopkg.in/macaron.v1#Context_ServeFileContent)
- 获取远程 IP 地址：[`ctx.RemoteAddr`](https://gowalker.org/gopkg.in/macaron.v1#Context_RemoteAddr)

## 路由日志

该服务可以通过函数  [`macaron.Logger`](https://gowalker.org/gopkg.in/macaron.v1#Logger) 来注入。该服务主要负责应用的路由日志。

使用方法：

```go
package main

import "gopkg.in/macaron.v1"

func main() {
	m := macaron.New()
	m.Use(macaron.Logger())
	// ...
}
```

**备注** 当您使用 [`macaron.Classic`](https://gowalker.org/gopkg.in/macaron.v1#Classic) 时，该服务会被自动注入。

从 [Peach](https://github.com/peachdocs/peach) 项目中提取的样例输出：

```
[Macaron] Started GET /docs/middlewares/core.html for [::1]
[Macaron] Completed /docs/middlewares/core.html 200 OK in 2.114956ms
```

## 容错恢复

该服务可以通过函数 [`macaron.Recovery`](https://gowalker.org/gopkg.in/macaron.v1#Recovery) 来注入。该服务主要负责在应用发生恐慌（panic）时进行恢复。

使用方法：

```go
package main

import "gopkg.in/macaron.v1"

func main() {
	m := macaron.New()
	m.Use(macaron.Recovery())
	// ...
}
```

**备注** 当您使用 [`macaron.Classic`](https://gowalker.org/gopkg.in/macaron.v1#Classic) 时，该服务会被自动注入。

## 静态文件

该服务可以通过函数 [`macaron.Static`](https://gowalker.org/gopkg.in/macaron.v1#Static) 来注入。该服务主要负责应用静态资源的服务，当您的应用拥有多个静态目录时，可以对其进行多次注入。

使用方法：

```go
package main

import "gopkg.in/macaron.v1"

func main() {
	m := macaron.New()
	m.Use(macaron.Static("public"))
	m.Use(macaron.Static("assets"))
	// ...
}
```

**备注** 当您使用 [`macaron.Classic`](https://gowalker.org/gopkg.in/macaron.v1#Classic) 时，该服务会以 `public` 为静态目录被自动注入。

默认情况下，当您请求一个目录时，该服务不会列出目录下的文件，而是去寻找 `index.html` 文件。

从 [Peach](https://github.com/peachdocs/peach) 项目中提取的样例输出：

```
[Macaron] Started GET /css/prettify.css for [::1]
[Macaron] [Static] Serving /css/prettify.css
[Macaron] Completed /css/prettify.css 304 Not Modified in 97.584us
[Macaron] Started GET /imgs/macaron.png for [::1]
[Macaron] [Static] Serving /imgs/macaron.png
[Macaron] Completed /imgs/macaron.png 304 Not Modified in 123.211us
[Macaron] Started GET /js/gogsweb.min.js for [::1]
[Macaron] [Static] Serving /js/gogsweb.min.js
[Macaron] Completed /js/gogsweb.min.js 304 Not Modified in 47.653us
[Macaron] Started GET /css/main.css for [::1]
[Macaron] [Static] Serving /css/main.css
[Macaron] Completed /css/main.css 304 Not Modified in 42.58us
```

### 使用示例

假设您的应用拥有以下目录结构：

```
public/
	|__ html
			|__ index.html
	|__ css/
			|__ main.css
```

响应结果：

请求 URL|匹配文件
-----------|----------
`/html/main.html`|匹配失败
`/html/`|index.html
`/css/main.css`|main.css

### 自定义选项

该服务允许接受第二个参数来进行自定义选项操作（[`macaron.StaticOptions`](https://gowalker.org/gopkg.in/macaron.v1#StaticOptions)）：

```go
package main

import "gopkg.in/macaron.v1"

func main() {
	m := macaron.New()
	m.Use(macaron.Static("public",
		macaron.StaticOptions{
			// 请求静态资源时的 URL 前缀，默认没有前缀
			Prefix: "public",
			// 禁止记录静态资源路由日志，默认为不禁止记录
			SkipLogging: true,
			// 当请求目录时的默认索引文件，默认为 "index.html"
			IndexFile: "index.html",
			// 用于返回自定义过期响应头，默认为不设置
			// https://developers.google.com/speed/docs/insights/LeverageBrowserCaching
			Expires: func() string { 
				return time.Now().Add(24 * 60 * time.Minute).UTC().Format("Mon, 02 Jan 2006 15:04:05 GMT")
			},
		}))
	// ...
}
```

### 注册多个静态处理器

如果您需要一次注册多个静态处理器，可以使用方法 [`macaron.Statics`](https://gowalker.org/gopkg.in/macaron.v1#Statics) 来简化您的工作。

使用方法：

```go
// ...
m.Use(macaron.Statics(macaron.StaticOptions{}, "public", "views"))
// ...
```

这样，就可以同时注册 `public` 和 `views` 为静态目录了。

## 其它服务

### 全局日志

该服务通过类型 [`*log.Logger`](http://gowalker.org/log#Logger) 来体现。该服务为可选，只是为没有日志器的应用提供一定的便利。

使用方法：

```go
package main

import (
	"log"

	"gopkg.in/macaron.v1"
)

func main() {
	m := macaron.Classic()
	m.Get("/", myHandler)
	m.Run()
}

func myHandler(ctx *macaron.Context, logger *log.Logger) string {
	logger.Println("the request path is: " + ctx.Req.RequestURI)
	return "the request path is: " + ctx.Req.RequestURI
}
```

**备注** 所有 [Macaron 实例](../intro/core_concepts#macaron-%E5%AE%9E%E4%BE%8B) 都会自动注册该服务。

### 响应流

该服务通过类型 [`http.ResponseWriter`](http://gowalker.org/net/http#ResponseWriter) 来体现。该服务为可选，一般情况下可直接使用 `*macaron.Context.Resp`。

使用方法：

```go
package main

import (
	"gopkg.in/macaron.v1"
)

func main() {
	m := macaron.Classic()
	m.Get("/", myHandler)
	m.Run()
}

func myHandler(ctx *macaron.Context) {
	ctx.Resp.Write([]byte("the request path is: " + ctx.Req.RequestURI))
}
```

**备注** 所有 [Macaron 实例](../intro/core_concepts#macaron-%E5%AE%9E%E4%BE%8B) 都会自动注册该服务。

### 请求对象

该服务通过类型 [`*http.Request`](http://gowalker.org/net/http#Request) 来体现。该服务为可选，一般情况下可直接使用 `*macaron.Context.Req`。

除此之外，该服务还提供了 3 个便利的方法来获取请求体：

- [`*macaron.Context.Req.Body().String()`](https://gowalker.org/gopkg.in/macaron.v1#RequestBody_String)：获取 `string` 类型的请求体
- [`*macaron.Context.Req.Body().Bytes()`](https://gowalker.org/gopkg.in/macaron.v1#RequestBody_Bytes)：获取 `[]byte` 类型的请求体
- [`*macaron.Context.Req.Body().ReadCloser()`](https://gowalker.org/gopkg.in/macaron.v1#RequestBody_ReadCloser)：获取 `io.ReadCloser` 类型的请求体

使用方法：

```go
package main

import (
	"gopkg.in/macaron.v1"
)

func main() {
	m := macaron.Classic()
	m.Get("/body1", func(ctx *macaron.Context) {
		reader, err := ctx.Req.Body().ReadCloser()
		// ...
	})
	m.Get("/body2", func(ctx *macaron.Context) {
		data, err := ctx.Req.Body().Bytes()
		// ...
	})
	m.Get("/body3", func(ctx *macaron.Context) {
		data, err := ctx.Req.Body().String()
		// ...
	})
	m.Run()
}
```

需要注意的是，请求体在每个请求中只能被读取一次。

有时您需要传递类型为 [`*http.Request`](http://gowalker.org/net/http#Request) 的参数，则应该使用 `*macaron.Context.Req.Request`。

**备注** 所有 [Macaron 实例](../intro/core_concepts#macaron-%E5%AE%9E%E4%BE%8B) 都会自动注册该服务。
