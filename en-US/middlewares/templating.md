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
		// Directory to load templates.
		Directory: "templates",
		// Extensions to parse template files from.
		Extensions: []string{".tmpl", ".html"},
		// Funcs is a slice of FuncMaps to apply to the template upon compilation.
		Funcs: []template.FuncMap{map[string]interface{}{
			"AppName": func() string {
				return "Macaron"
			},
			"AppVer": func() string {
				return "1.0.0"
			},
		},
		// Delims sets the action delimiters to the specified strings.
		Delims: macaron.Delims{"{{", "}}"},
		// Appends the given charset to the Content-Type header.
		Charset: "UTF-8",
		// Outputs human readable JSON.
		IndentJSON: true,
		// Outputs human readable XML.
		
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

### Qucik summary on rendering HTML

As you can see, the only difference between two templating engines to render HTML is the syntax of template files, in the code level, they are exactly the same.