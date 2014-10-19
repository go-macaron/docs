---
root: false
name: 核心概念
sort: 1
---

# Macaron 核心概念

## 经典 Macaron

为了更快速的启用 Macaron, [macaron.Classic()](https://gowalker.org/github.com/Unknwon/macaron#Classic) 提供了一些默认的组件以方便 Web 开发:

```go
  m := macaron.Classic()
  // ... 可以在这里使用中间件和注册路由
  m.Run()
```

下面是经典 Macaron 已经包含的功能 [`macaron.Classic`](https://gowalker.org/github.com/Unknwon/macaron#Classic)：

- 请求/响应日志 - [`macaron.Logger`](../middlewares/core#routing-logger)
- 容错恢复 - [`macaron.Recovery`](../middlewares/core#panic-recovery)
- 静态文件服务 - [`macaron.Static`](../middlewares/core#static-files)

## Macaron 实例

任何类型为 [`macaron.Macaron`](https://gowalker.org/github.com/Unknwon/macaron#Macaron) 的对象都可以被认为是 Macaron 的实例，您可以在单个应用中使用任意数量的 Macaron 实例。

## 处理器

处理器是 Macaron 的灵魂和核心所在. 一个处理器基本上可以是任何的函数:

```go
m.Get("/", func() {
	println("hello world")
})
```

### 返回值

当一个处理器返回结果的时候, Macaron 将会把返回值作为字符串写入到当前的 [http.ResponseWriter](http://gowalker.org/net/http#ResponseWriter) 里面：

```go
m.Get("/", func() string {
	return "hello world" // HTTP 200 : "hello world"
})
```

另外你也可以选择性的返回状态码:

```go
m.Get("/", func() (int, string) {
	return 418, "i'm a teapot" // HTTP 418 : "i'm a teapot"
})
```

### 服务注入

处理器是通过反射来调用的，Macaron 通过 [依赖注入](http://baike.baidu.com/view/1486379.htm?from_id=5177233&type=syn&fromtitle=%E4%BE%9D%E8%B5%96%E6%B3%A8%E5%85%A5&fr=aladdin) 来为处理器注入参数列表。 **这样使得 Macaron 与 Go 语言的 [`http.HandlerFunc`](https://gowalker.org/net/http#HandlerFunc) 接口完全兼容**。

如果你加入一个参数到你的处理器, Macaron 将会搜索它参数列表中的服务，并且通过类型判断来解决依赖关系：

```go
m.Get("/", func(resp http.ResponseWriter, req *http.Request) { 
	// resp 和 req 是由 Macaron 默认注入的服务
	resp.WriteHeader(200) // HTTP 200
})
```

下面的这些服务已经被包含在经典 Macaron 中（[macaron.Classic](https://gowalker.org/github.com/Unknwon/macaron#Classic)）：

- [*macaron.Context](../middlewares/core#context) - HTTP 请求上下文
- [*log.Logger](http://gowalker.org/log#Logger) - Macaron 全局日志器
- [http.ResponseWriter](http://gowalker.org/net/http/#ResponseWriter) - HTTP 响应流接口
- [*http.Request](http://gowalker.org/net/http/#Request) - HTTP 请求对象

## Macaron 环境变量

一些 Macaron 处理器依赖 `macaron.Env` 全局变量为开发模式和部署模式表现出不同的行为，不过更建议使用环境变量 `MACARON_ENV=production` 来指示当前的模式为部署模式。