---
root: false
name: Templating
sort: 2
---

# Templating

There are two official middlewares built for templating for your Macaron application currently, which are [`macaron.Renderer`](https://gowalker.org/github.com/Unknwon/macaron#Renderer) and [`pongo2.Pongoer`](https://gowalker.org/github.com/macaron-contrib/pongo2#Pongoer).

You're free to choose them to use, but normally, one Macaron [Instance](../intro/core_concepts#instances) only uses one templating engine.

Common behaviors:

- Both of them are supporting render XML, JSON and raw content as response, the only difference between them is the way to render HTML.
- Both of them use `templates` as default template file directory.
- Both of them use `.tmpl` and `.html` as default template file extensions.
- Both of them support cache template files in memory when `macaron.Env == macaron.PROD`.

## Render HTML

### Go Templating Engine

This service can be injected by function [`macaron.Renderer`](https://gowalker.org/github.com/Unknwon/macaron#Renderer) and is represented by type [`macaron.Render`](https://gowalker.org/github.com/Unknwon/macaron#Render). It is optional to use, normally, you should use `*macaron.Context.Render`.This service uses Go built-in templating engine to render your HTML. If you want to know about details of how it works, please see [documents](https://gowalker.org/html/template).

#### Example

Suppose you have following directory structure:

```
main/
	|__ main.go
	|__ templates/
			|__ hello.tmpl
```

hello.tmpl:

```html
<h1>Hello {{.Name}}</h1>
```

main.go:

```go
package main

import "github.com/Unknwon/macaron"

func main() {
	m := macaron.Classic()
	m.Use(macaron.Renderer())
	
	m.Get("/", func(ctx *macaron.Context) {
		ctx.Data["Name"] = "jeremy"
		ctx.HTML(200, "hello") // 200 is the response code.
	})
	
	m.Run()
}
```

#### Options

This service also accepts one argument for custom options([`macaron.RenderOptions`](https://gowalker.org/github.com/Unknwon/macaron#RenderOptions)):

```go
package main

import "github.com/Unknwon/macaron"

func main() {
	m := macaron.Classic()
	m.Use(macaron.Renderer(macaron.RenderOptions{
		// Directory to load templates. Default is "templates".
		Directory: "templates",
		// Extensions to parse template files from. Defaults are [".tmpl", ".html"].
		Extensions: []string{".tmpl", ".html"},
		// Funcs is a slice of FuncMaps to apply to the template upon compilation. Default is [].
		Funcs: []template.FuncMap{map[string]interface{}{
			"AppName": func() string {
				return "Macaron"
			},
			"AppVer": func() string {
				return "1.0.0"
			},
		},
		// Delims sets the action delimiters to the specified strings. Defaults are ["{{", "}}"].
		Delims: macaron.Delims{"{{", "}}"},
		// Appends the given charset to the Content-Type header. Default is "UTF-8".
		Charset: "UTF-8",
		// Outputs human readable JSON. Default is false.
		IndentJSON: true,
		// Outputs human readable XML. Default is false.
		IndentXML: true,
		// Prefixes the JSON output with the given bytes. Default is no prefix.
		PrefixJSON: []byte("macaron"),
		// Prefixes the XML output with the given bytes. Default is no prefix.
		PrefixXML: []byte("macaron"),
		// Allows changing of output to XHTML instead of HTML. Default is "text/html".
		HTMLContentType: "text/html",
	}))		
	// ...
}
```

### Pongo2 Templating Engine

This service can be injected by function [`pongo2.Pongoer`](https://gowalker.org/github.com/macaron-contrib/pongo2#Pongoer) and is represented by type [`macaron.Render`](https://gowalker.org/github.com/Unknwon/macaron#Render). It is optional to use, normally, you should use `*macaron.Context.Render`.This service uses Pongo2 templating engine to render your HTML. If you want to know about details of how it works, please see [documents](https://github.com/flosch/pongo2).

#### Example

Suppose you have following directory structure:

```
main/
	|__ main.go
	|__ templates/
			|__ hello.tmpl
```

hello.tmpl:

```html
<h1>Hello {{Name}}</h1>
```

main.go:

```go
package main

import (
	"github.com/Unknwon/macaron"
	"github.com/macaron-contrib/pongo2"
)

func main() {
	m := macaron.Classic()
	m.Use(pongo2.Pongoer())
	
	m.Get("/", func(ctx *macaron.Context) {
		ctx.Data["Name"] = "jeremy"
		ctx.HTML(200, "hello") // 200 is the response code.
	})
	
	m.Run()
}
```

#### Options

This service also accepts one argument for custom options([`pongo2.Options`](https://gowalker.org/github.com/macaron-contrib/pongo2#Options)):

```go
package main

import (
	"github.com/Unknwon/macaron"
	"github.com/macaron-contrib/pongo2"
)

func main() {
	m := macaron.Classic()
	m.Use(pongo2.Pongoer(pongo2.Options{
		// Directory to load templates. Default is "templates".
		Directory: "templates",
		// Extensions to parse template files from. Defaults are [".tmpl", ".html"].
		Extensions: []string{".tmpl", ".html"},
		// Appends the given charset to the Content-Type header. Default is "UTF-8".
		Charset: "UTF-8",
		// Outputs human readable JSON. Default is false.
		IndentJSON: true,
		// Outputs human readable XML. Default is false.
		IndentXML: true,
		// Allows changing of output to XHTML instead of HTML. Default is "text/html".
		HTMLContentType: "text/html",
	}))		
	// ...
}
```

### Qucik summary on rendering HTML

As you can see, the only difference between two templating engines to render HTML is the syntax of template files, in the code level, they are exactly the same.

If you just want to get results of rendered HTML, call method `*macaron.Context.Render.HTMLString`:

```go
package main

import "github.com/Unknwon/macaron"

func main() {
    m := macaron.Classic()
    m.Use(macaron.Renderer())
    
    m.Get("/", func(ctx *macaron.Context) {
        ctx.Data["Name"] = "jeremy"
        output, err := ctx.HTMLString("hello")
        // Do other operations
    })
    
    m.Run()
}
```

## Render XML, JSON and raw content

It is fairly easy to render XML, JSON and raw content compare to HTML.

```go
package main

import (
	"github.com/Unknwon/macaron"
)

type Person struct {
	Name string
	Age  int
	Sex  string
}

func main() {
	m := macaron.Classic()
	m.Use(macaron.Renderer())

	m.Get("/xml", func(ctx *macaron.Context) {
		p := Person{"Unknwon", 21, "male"}
		ctx.XML(200, &p)
	})
	m.Get("/json", func(ctx *macaron.Context) {
		p := Person{"Unknwon", 21, "male"}
		ctx.JSON(200, &p)
	})
	m.Get("/raw", func(ctx *macaron.Context) {
		ctx.RawData(200, []byte("raw data goes here"))
	})

	m.Run()
}
```

## Response status, error and redirect

To response status, error and redirect:

```go
package main

import (
	"github.com/Unknwon/macaron"
)

func main() {
	m := macaron.Classic()
	m.Use(macaron.Renderer())

	m.Get("/status", func(ctx *macaron.Context) {
		ctx.Status(403)
	})
	m.Get("/error", func(ctx *macaron.Context) {
		ctx.Error(500, "Internal Server Error")
	})
	m.Get("/redirect", func(ctx *macaron.Context) {
		ctx.Redirect("/") // The second argument is response code, default is 302.
	})

	m.Run()
}
```

## Change template path at runtime

In case you want to change your template path at runtime, call method `*macaron.Context.SetTemplatePath`. Note that this operation applies to global, not just current request.

### Example

Suppose you have following directory structure:

```
main/
	|__ main.go
	|__ templates/
			|__ hello.tmpl
	|__ templates2/
			|__ hello.tmpl
```

templates/hello.tmpl:

```html
<h1>Hello {{.Name}}</h1>
```

templates2/hello.tmpl:

```html
<h1>What's up, {{.Name}}</h1>
```

main.go:

```go
package main

import (
	"github.com/Unknwon/macaron"
)

func main() {
	m := macaron.Classic()
	m.Use(macaron.Renderer())

	m.Get("/old", func(ctx *macaron.Context) {
		ctx.Data["Name"] = "Unknwon"
		ctx.HTML(200, "hello")
		ctx.SetTemplatePath("templates2")
	})
	m.Get("/new", func(ctx *macaron.Context) {
		ctx.Data["Name"] = "Unknwon"
		ctx.HTML(200, "hello")
	})

	m.Run()
}
```

When you first request `/old`, the response will be `<h1>Hello Unknwon</h1>`, right after response, the template path has been changed to `template2`. So when you request `/new`, the response will be `<h1>What's up, Unknwon</h1>`.