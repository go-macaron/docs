---
root: false
name: 自定义服务
sort: 0
---

# 自定义服务

服务即是被注入到处理器中的参数. 你可以映射一个服务到 *全局* 或者 *请求* 的级别.

#### 全局映射

因为 Macaron 实现了 `inject.Injector` 的接口, 那么映射成为一个服务就非常简单:

```go
db := &MyDatabase{}
m := martini.Classic()
m.Map(db) // the service will be available to all handlers as *MyDatabase
// ...
m.Run()
```

#### 请求级别的映射

映射在请求级别的服务可以用 [*macaron.Context](https://gowalker.org/github.com/Unknwon/macaron#Context) 来完成:

```go
func MyCustomLoggerHandler(ctx *macaron.Context) {
	logger := &MyCustomLogger{ctx.Req}
	ctx.Map(logger) // mapped as *MyCustomLogger
}
```

#### 映射值到接口

关于服务最强悍的地方之一就是它能够映射服务到接口. 例如说, 假设你想要覆盖 [http.ResponseWriter](http://gowalker.org/net/http#ResponseWriter) 成为一个对象, 那么你可以封装它并包含你自己的额外操作, 你可以如下这样来编写你的处理器:

```go
func WrapResponseWriter(ctx *macaron.Context) {
	rw := NewSpecialResponseWriter(ctx.Resp)
	// override ResponseWriter with our wrapper ResponseWriter
	ctx.MapTo(rw, (*http.ResponseWriter)(nil)) 
}
```
