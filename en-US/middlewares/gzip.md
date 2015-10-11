---
name: Gzip
---

# Gzip

Middleware gzip provides compress to responses for Macaron [Instances](../intro/core_concepts#instances). Make sure to register it before other middlewares that write content to response.

- [GitHub](https://github.com/go-macaron/gzip)
- [API Reference](https://gowalker.org/github.com/go-macaron/gzip)

## Installation

```sh
go get github.com/github.com/go-macaron/gzip
```

## Usage

```go
package main

import (
	"gopkg.in/macaron.v1"
	"github.com/go-macaron/gzip"
)

func main() {
	m := macaron.Classic()
	m.Use(gzip.Gziper())
	// Register routers.
	m.Run()
}
```

In this case, the static files will not be compressed by Gzip, to compress them:

```go
package main

import (
	"gopkg.in/macaron.v1"
	"github.com/go-macaron/gzip"
)

func main() {
	m := macaron.New()
	m.Use(macaron.Logger())
	m.Use(macaron.Recovery())
	m.Use(gzip.Gziper())
	m.Use(macaron.Static("public"))
	// Register routers.
	m.Run()
}
```

Or you can choose to only compress a group of routes responses:

```go
// ...

func main() {
	m := macaron.Classic()
	m.Group("/gzip", func() {
		// ...
	}, gzip.Gziper())
	// ...
	m.Run()
}
```

## Options

This service comes with a variety of configuration options([`gzip.Options`](https://gowalker.org/github.com/go-macaron/gzip#Options)):

```go
// ...
m.Use(gzip.Gziper(gzip.Options{
	// Compression level. Can be DefaultCompression(-1), ConstantCompression(-2)
	// or any integer value between BestSpeed(1) and BestCompression(9) inclusive.
	CompressionLevel: 4,
}))
// ...
```