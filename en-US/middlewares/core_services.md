---
name: Core Services
---

# Core Services

By default, Macaron injects some services to power your application, those services are known as **core services**, which means you can directly use them as handler arguments without any additional work.

## Context

This service is represented by type [`*macaron.Context`](https://gowalker.org/gopkg.in/macaron.v1#Context). This is the very core serivce for everything you do upon Macaron. It contains all the information you need for request, response, templating, data store, and inject or retrieve other services.

To use it:

```go
package main

import "gopkg.in/macaron.v1"

func Home(ctx *macaron.Context) {
	// ...
}
```

### Next()

Method [`Context.Next`](https://gowalker.org/gopkg.in/macaron.v1#Context_Next) is an optional feature that Middleware Handlers can call to yield the until after the other Handlers have been executed. This works really well for any operations that must happen after an HTTP request:

```go
// log before and after a request
m.Use(func(ctx *macaron.Context, log *log.Logger){
	log.Println("before a request")

	ctx.Next()

	log.Println("after a request")
})
```

### Cookie

The very basic usage of cookie is just:

- [`*macaron.Context.SetCookie`](https://gowalker.org/gopkg.in/macaron.v1#Context_SetCookie)
- [`*macaron.Context.GetCookie`](https://gowalker.org/gopkg.in/macaron.v1#Context_GetCookie), [`*macaron.Context.GetCookieInt`](https://gowalker.org/gopkg.in/macaron.v1#Context_GetCookieInt), [`*macaron.Context.GetCookieInt64`](https://gowalker.org/gopkg.in/macaron.v1#Context_GetCookieInt64), [`*macaron.Context.GetCookieFloat64`](https://gowalker.org/gopkg.in/macaron.v1#Context_GetCookieFloat64)

To use them:

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

Use following arguments order to set more properties: `SetCookie(<name>, <value>, <max age>, <path>, <domain>, <secure>, <http only>)`.

For example, the most advanced usage would be: `SetCookie("user", "unknwon", 999, "/", "localhost", true, true)`.

Note that order is fixed.

There are also more secure cookie support. First, you need to call [`macaron.SetDefaultCookieSecret`](https://gowalker.org/gopkg.in/macaron.v1#Macaron_SetDefaultCookieSecret), then use it by calling:

- [`*macaron.Context.SetSecureCookie`](https://gowalker.org/gopkg.in/macaron.v1#Context_SetSecureCookie)
- [`*macaron.Context.GetSecureCookie`](https://gowalker.org/gopkg.in/macaron.v1#Context_GetSecureCookie)

These two methods uses default secret string you set globally to encode and decode values.

To use them:

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

For people who wants even more secure cookies that change secret string every time, just use:

- [`*macaron.Context.SetSuperSecureCookie`](https://gowalker.org/gopkg.in/macaron.v1#Context_SetSuperSecureCookie)
- [`*macaron.Context.GetSuperSecureCookie`](https://gowalker.org/gopkg.in/macaron.v1#Context_GetSuperSecureCookie)

To use them:

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

### Other Helper methods

- To set/get URL parameters: [`ctx.SetParams`](https://gowalker.org/gopkg.in/macaron.v1#Context_SetParams) / [`ctx.Params`](https://gowalker.org/gopkg.in/macaron.v1#Context_Params), [`ctx.ParamsEscape`](https://gowalker.org/gopkg.in/macaron.v1#Context_ParamsEscape), [`ctx.ParamsInt`](https://gowalker.org/gopkg.in/macaron.v1#Context_ParamsInt), [`ctx.ParamsInt64`](https://gowalker.org/gopkg.in/macaron.v1#Context_ParamsInt64), [`ctx.ParamsFloat64`](https://gowalker.org/gopkg.in/macaron.v1#Context_ParamsFloat64)
- To get query parameters: [`ctx.Query`](https://gowalker.org/gopkg.in/macaron.v1#Context_ctx.Query), [`ctx.QueryEscape`](https://gowalker.org/gopkg.in/macaron.v1#Context_ctx.QueryEscape), [`ctx.QueryInt`](https://gowalker.org/gopkg.in/macaron.v1#Context_ctx.QueryInt), [`ctx.QueryInt64`](https://gowalker.org/gopkg.in/macaron.v1#Context_ctx.QueryInt64), [`ctx.QueryFloat64`](https://gowalker.org/gopkg.in/macaron.v1#Context_ctx.QueryFloat64), [`ctx.QueryStrings`](https://gowalker.org/gopkg.in/macaron.v1#Context_ctx.QueryStrings), [`ctx.QueryTrim`](https://gowalker.org/gopkg.in/macaron.v1#Context_QueryTrim)
- To serve content or file: [`ctx.ServeContent`](https://gowalker.org/gopkg.in/macaron.v1#Context_ServeContent), [`ctx.ServeFile`](https://gowalker.org/gopkg.in/macaron.v1#Context_ServeFile), [`ctx.ServeFileContent`](https://gowalker.org/gopkg.in/macaron.v1#Context_ServeFileContent)
- To get remote IP address: [`ctx.RemoteAddr`](https://gowalker.org/gopkg.in/macaron.v1#Context_RemoteAddr)

## Router Logger

This service can be injected by function  [`macaron.Logger`](https://gowalker.org/gopkg.in/macaron.v1#Logger). It is responsible for your application routing log.

To use it:

```go
package main

import "gopkg.in/macaron.v1"

func main() {
	m := macaron.New()
	m.Use(macaron.Logger())
	// ...
}
```

**Note** this service is injected automatically when you use [`macaron.Classic`](https://gowalker.org/gopkg.in/macaron.v1#Classic).

Sample output take from [Peach](https://github.com/peachdocs/peach):

```
[Macaron] Started GET /docs/middlewares/core.html for [::1]
[Macaron] Completed /docs/middlewares/core.html 200 OK in 2.114956ms
```

## Panic Recovery

This service can be injected by function [`macaron.Recovery`](https://gowalker.org/gopkg.in/macaron.v1#Recovery). It is responsible for recovering your application when panic haapens.

To use it:

```go
package main

import "gopkg.in/macaron.v1"

func main() {
	m := macaron.New()
	m.Use(macaron.Recovery())
	// ...
}
```

**Note** this service is injected automatically when you use [`macaron.Classic`](https://gowalker.org/gopkg.in/macaron.v1#Classic).

## Static Files

This service can be injected by function [`macaron.Static`](https://gowalker.org/gopkg.in/macaron.v1#Static). It is responsible for serving static resources of your application, it can be injected as many times as you want if you have multiple static directories.

To use it:

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

**Note** this service is injected automatically with directory `public` when you use [`macaron.Classic`](https://gowalker.org/gopkg.in/macaron.v1#Classic).

By default, when you try to request a directory, this service will not list directory files. Instead, it tries to find the `index.html` file.

Sample output take from [Peach](https://github.com/peachdocs/peach):

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

### Example

Suppose you have following directory structure:

```
public/
	|__ html
			|__ index.html
	|__ css/
			|__ main.css
```

Results:

Request URL|Match File
-----------|----------
`/html/main.html`|None
`/html/`|index.html
`/css/main.css`|main.css

### Options

This service also accepts second argument for custom options([`macaron.StaticOptions`](https://gowalker.org/gopkg.in/macaron.v1#StaticOptions)):

```go
package main

import "gopkg.in/macaron.v1"

func main() {
	m := macaron.New()
	m.Use(macaron.Static("public",
		macaron.StaticOptions{
			// Prefix is the optional prefix used to serve the static directory content. Default is empty string.
			Prefix: "public",
			// SkipLogging will disable [Static] log messages when a static file is served. Default is false.
			SkipLogging: true,
			// IndexFile defines which file to serve as index if it exists. Default is "index.html".
			IndexFile: "index.html",
			// Expires defines which user-defined function to use for producing a HTTP Expires Header. Default is nil.
			// https://developers.google.com/speed/docs/insights/LeverageBrowserCaching
			Expires: func() string { 
				return time.Now().Add(24 * 60 * time.Minute).UTC().Format("Mon, 02 Jan 2006 15:04:05 GMT")
			},
		}))
	// ...
}
```

### Multiple Static Handlers

In case you have multiple static directories, there is one helper function [`macaron.Statics`](https://gowalker.org/gopkg.in/macaron.v1#Statics) to make your life easier.

To use it:

```go
// ...
m.Use(macaron.Statics(macaron.StaticOptions{}, "public", "views"))
// ...
```

This will register both `public` and `views` as static directories.

## Others Services

### Global Logger

This service is represented by type [`*log.Logger`](http://gowalker.org/log#Logger). It is optional to use, but for convenience when you do not have a global level logger.

To use it:

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

**Note** this service is injected automatically for every Macaron [Instance](../intro/core_concepts#instances).

### Response Stream

This service is represented by type [`http.ResponseWriter`](http://gowalker.org/net/http#ResponseWriter). It is optional to use, normally, you should use `*macaron.Context.Resp`.

To use it:

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

**Note** this service is injected automatically for every Macaron [Instance](../intro/core_concepts#instances).

### Request Object

This service is represented by type [`*http.Request`](http://gowalker.org/net/http#Request). It is optional to use, normally, you should use `*macaron.Context.Req`.

Besides, this service provides three methods to help you easily retrieve request body:

- [`*macaron.Context.Req.Body().String()`](https://gowalker.org/gopkg.in/macaron.v1#RequestBody_String): get request body in `string` type
- [`*macaron.Context.Req.Body().Bytes()`](https://gowalker.org/gopkg.in/macaron.v1#RequestBody_Bytes): get request body in `[]byte` type
- [`*macaron.Context.Req.Body().ReadCloser()`](https://gowalker.org/gopkg.in/macaron.v1#RequestBody_ReadCloser): get request body in `io.ReadCloser` type

To use them:

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

Notice that request body can only be read once.

Sometimes you need to pass type [`*http.Request`](http://gowalker.org/net/http#Request) as an argument, you should use `*macaron.Context.Req.Request`.

**Note** this service is injected automatically for every Macaron [Instance](../intro/core_concepts#instances).
