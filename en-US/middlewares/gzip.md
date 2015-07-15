---
root: false
name: Gzip
sort: 1
---

# Gzip

Middleware Gzip provides Gzip compress to responses. Make sure to register it before other middlewares that write content to response.

## Usage

```go
package main

import "github.com/Unknwon/macaron"

func main() {
	m := macaron.Classic()
	m.Use(macaron.Gziper())
	// Register routers.
	m.Run()
}
```

In this case, the static files will not be compressed by Gzip, to compress them:

```go
package main

import "github.com/Unknwon/macaron"

func main() {
	m := macaron.New()
	m.Use(macaron.Logger())
	m.Use(macaron.Recovery())
	m.Use(macaron.Gziper())
	m.Use(macaron.Static("public"))
	// Register routers.
	m.Run()
}
```

## Options

`macaron.Gziper` comes with a variety of configuration options([`macaron.GzipOptions`](https://gowalker.org/github.com/Unknwon/macaron#GzipOptions)):

```go
// ...
m.Use(macaron.Gziper(macaron.GzipOptions{
	// Compression level. Can be DefaultCompression(-1) or any integer value between BestSpeed(1) and BestCompression(9) inclusive.
	CompressionLevel: 1,
}))
```