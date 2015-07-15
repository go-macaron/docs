---
root: false
name: Gzip
sort: 1
---

# Gzip

中间件 Gzip 可以为响应内容提供 Gzip 压缩。请确保在其它会向响应流写入内容的中间件之前注册该服务。

## 使用示例

```go
package main

import "github.com/Unknwon/macaron"

func main() {
	m := macaron.Classic()
	m.Use(macaron.Gziper())
	// 注册路由
	m.Run()
}
```

在这个例子中，静态资源不会被 Gzip 压缩，如果想压缩它们，则可以使用以下方法：

```go
package main

import "github.com/Unknwon/macaron"

func main() {
	m := macaron.New()
	m.Use(macaron.Logger())
	m.Use(macaron.Recovery())
	m.Use(macaron.Gziper())
	m.Use(macaron.Static("public"))
	// 注册路由
	m.Run()
}
```

## 自定义选项

该服务允许接受一个参数来进行自定义选项（[`macaron.GzipOptions`](https://gowalker.org/github.com/Unknwon/macaron#GzipOptions)）：

```go
// ...
m.Use(macaron.Gziper(macaron.GzipOptions{
	// 压缩级别，可以是 DefaultCompression（-1），或介于包括 BestSpeed（1） 和 BestCompression（9） 在内，这两者之间的任意整数
	CompressionLevel: 1,
}))
```