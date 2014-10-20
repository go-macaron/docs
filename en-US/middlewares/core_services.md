---
root: false
name: Core Services
sort: 0
---

# Core Services

By default, Macaron injects some services to power your application, those services are known as **core services**, which means you can directly use them as handler arguments without any additional work.

## Context

This service is represented by type [`*macaron.Context`](https://gowalker.org/github.com/Unknwon/macaron#Context). This is the very core serivce for everything you do upon Macaron. It contains all the information you need for request, response, templating, data store, and inject or retrieve other services.

To use it:

```go
package main

import "github.com/Unknwon/macaron"

func Home(ctx *macaron.Context) {
	// ...
}
```

## Routing Logger

This service can be injected by function  [`macaron.Logger`](https://gowalker.org/github.com/Unknwon/macaron#Logger). It is responsible for your application routing log.

To use it:

```go
package main

import "github.com/Unknwon/macaron"

func main() {
	m := macaron.New()
	m.Use(macaron.Logger())
	// ...
}
``` 

**Note** this service is injected automatically when you use [`macaron.Classic`](https://gowalker.org/github.com/Unknwon/macaron#Classic).

Sample output take from [Gogs Web](https://github.com/gogits/gogs):

```
[Macaron] Started GET /docs/middlewares/core.html for [::1]
[Macaron] Completed /docs/middlewares/core.html 200 OK in 2.114956ms
```

## Panic Recovery

This service can be injected by function [`macaron.Recovery`](https://gowalker.org/github.com/Unknwon/macaron#Recovery). It is responsible for recovering your application when panic haapens.

To use it:

```go
package main

import "github.com/Unknwon/macaron"

func main() {
	m := macaron.New()
	m.Use(macaron.Recovery())
	// ...
}
``` 

**Note** this service is injected automatically when you use [`macaron.Classic`](https://gowalker.org/github.com/Unknwon/macaron#Classic).

## Static Files

This service can be injected by function [`macaron.Static`](https://gowalker.org/github.com/Unknwon/macaron#Static). It is responsible for serving static resources of your application, it can be injected as many times as you want if you have multiple static directories.

To use it:

```go
package main

import "github.com/Unknwon/macaron"

func main() {
	m := macaron.New()
	m.Use(macaron.Static("public"))
	m.Use(macaron.Static("assets"))
	// ...
}
``` 

**Note** this service is injected automatically with directory `public` when you use [`macaron.Classic`](https://gowalker.org/github.com/Unknwon/macaron#Classic).

By default, when you try to request a directory, this service will not list directory files. Instead, it tries to find the `index.html` file.

Sample output take from [Gogs Web](https://github.com/gogits/gogs):

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

This service also accepts second argument for custom options([`macaron.StaticOptions`](https://gowalker.org/github.com/Unknwon/macaron#StaticOptions)):

```go
package main

import "github.com/Unknwon/macaron"

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
			Expires: func() string { return "max-age=0" },
		}))
	// ...
}
```

## Others Services

### Global Logger

This service is represented by type [`*log.Logger`](http://gowalker.org/log#Logger). It is optional to use, but for convenience when you do not have a global level logger. 

To use it:

```go
package main

import (
	"log"

	"github.com/Unknwon/macaron"
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

This service is represented by type [`http.ResponseWriter`](http://gowalker.org/net/http/#ResponseWriter). It is optional to use, normally, you should use `*macaron.Context.Resp`.

To use it:

```go
package main

import (
	"github.com/Unknwon/macaron"
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

This service is represented by type [`*http.Request`](http://gowalker.org/net/http/#Request). It is optional to use, normally, you should use `*macaron.Context.Req`.

Besides, this service provides three methods to help you easily retrieve request body:

- [`*macaron.Context.Req.Body().String()`](https://gowalker.org/github.com/Unknwon/macaron#RequestBody_String): get request body in `string` type
- [`*macaron.Context.Req.Body().Bytes()`](https://gowalker.org/github.com/Unknwon/macaron#RequestBody_Bytes): get request body in `[]byte` type
- [`*macaron.Context.Req.Body().ReadCloser()`](https://gowalker.org/github.com/Unknwon/macaron#RequestBody_ReadCloser): get request body in `io.ReadCloser` type

To use them:

```go
package main

import (
	"github.com/Unknwon/macaron"
)

func main() {
	m := macaron.Classic()
	m.Get("/body1", func(ctx *Context) {
		reader, err := ctx.Req.Body().ReadCloser()
		// ...
	})
	m.Get("/body2", func(ctx *Context) {
		data, err := ctx.Req.Body().Bytes()
		// ...
	})
	m.Get("/body3", func(ctx *Context) {
		data, err := ctx.Req.Body().String()
		// ...
	})
	m.Run()
}
```

Notice that request body can only be read once.

**Note** this service is injected automatically for every Macaron [Instance](../intro/core_concepts#instances).