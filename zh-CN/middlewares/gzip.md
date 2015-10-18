---
name: Gzip
---

# Gzip

中间件 gzip 为 [Macaron 实例](../intro/core_concepts#macaron-%E5%AE%9E%E4%BE%8B) 的响应内容提供 Gzip 压缩。请确保在其它会向响应流写入内容的中间件之前注册该服务。

- [GitHub](https://github.com/go-macaron/gzip)
- [API 文档](https://gowalker.org/github.com/go-macaron/gzip)

## 下载安装

```sh
go get github.com/go-macaron/gzip
```

## 使用示例

```go
package main

import (
	"github.com/go-macaron/gzip"
	"gopkg.in/macaron.v1"
)

func main() {
	m := macaron.Classic()
	m.Use(gzip.Gziper())
	// 注册路由
	m.Run()
}
```

在这个例子中，静态资源不会被 Gzip 压缩，如果想压缩它们，则可以使用以下方法：

```go
package main

import (
	"github.com/go-macaron/gzip"
	"gopkg.in/macaron.v1"
)

func main() {
	m := macaron.New()
	m.Use(macaron.Logger())
	m.Use(macaron.Recovery())
	m.Use(gzip.Gziper())
	m.Use(macaron.Static("public"))
	// 注册路由
	m.Run()
}
```

或者选择只压缩某一组路由的响应内容：

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

## 自定义选项

该服务允许接受一个参数来进行自定义选项（[`gzip.Options`](https://gowalker.org/github.com/go-macaron/gzip#Options)）：

```go
// ...
m.Use(gzip.Gziper(gzip.Options{
	// 压缩级别，可以是 DefaultCompression（-1）、ConstantCompression（-2）
	// 或介于包括 BestSpeed（1） 和 BestCompression（9） 在内，这两者之间的任意整数。
	// 默认为 4
	CompressionLevel: 4,
}))
```