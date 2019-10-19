# Macaron [![Build Status](https://travis-ci.org/go-macaron/macaron.svg?branch=v1)](https://travis-ci.org/go-macaron/macaron)

Package macaron is a high productive and modular web framework in Go. It takes basic ideology of [Martini](https://github.com/go-martini/martini) and extends in advance.

## Getting Started

The minimum requirement of Go is **1.6**.

To install Macaron:

	go get gopkg.in/macaron.v1

The very basic usage of Macaron:

```go
package main

import "gopkg.in/macaron.v1"

func main() {
	m := macaron.Classic()
	m.Get("/", func() string {
		return "Hello world!"
	})
	m.Run()
}
```

## Features

- Powerful routing with suburl.
- Flexible routes combinations.
- Unlimited nested group routers.
- Directly integrate with existing services.
- Dynamically change template files at runtime.
- Allow to use in-memory template and static files.
- Easy to plugin/unplugin features with modular design.
- Handy dependency injection powered by [inject](https://github.com/codegangsta/inject).
- Better router layer and less reflection make faster speed.

## Use Cases

- [Gogs](https://gogs.io): A painless self-hosted Git Service
- [Grafana](http://grafana.org/): The open platform for beautiful analytics and monitoring
- [Peach](https://peachdocs.org): A modern web documentation server
- [Go Walker](https://gowalker.org): Go online API documentation
- [Critical Stack Intel](https://intel.criticalstack.com/): A 100% free intel marketplace from Critical Stack, Inc.

## Quick Start

- New to Macaron? Let’s [get started](/docs/intro/getting_started)!
- [Middlewares](/docs/middlewares) that are built for Macaron.
- Have any questions? Maybe there are [answers](/docs/faqs) for you!
- If you think anything is not clear in the documentation, just [file an issue](https://github.com/go-macaron/docs/issues)!
