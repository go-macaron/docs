---
root: true
name: 常见问题
sort: 2
---

# 常见问题

### 如何集成到我已有的服务中？

每个 [Macaron 实例](/docs/intro/core_concepts#macaron-%E5%AE%9E%E4%BE%8B) 都实现了 [`http.Handler`](https://gowalker.org/net/http#Handler) 接口，因此可以很容易地将它们以子集的形式集成到已有服务中。例如，您可以将 Macaron 应用集成到 GAE 中：

```go
package hello

import (
	"net/http"
	
	"github.com/Unknwon/macaron"
)

func init() {
	m := macaron.Classic()
	m.Get("/", func() string {
		return "Hello world!"
	})
	http.Handle("/", m)
}
```

### 如何修改监听地址和端口？

Macaron 的 `Run` 函数会首先根据环境变量 `PORT` 和 `HOST` 来确定监听地址和端口。如果未找到相应设置，则会默认使用 [localhost:4000](http://localhost:4000)。如果您想要更加灵活便利的方案，可以使用 [`http.ListenAndServe`](https://gowalker.org/net/http#ListenAndServe) 函数来实现。

```go
m := macaron.Classic()
// ...
log.Fatal(http.ListenAndServe(":8080", m))
```

或者以下方式：

- `m.Run("0.0.0.0")`，监听在 `0.0.0.0:4000`
- `m.Run(8080)`，监听在 `0.0.0.0:8080`
- `m.Run("0.0.0.0", 8080)`，监听在 `0.0.0.0:8080`

### 除了注入服务以外，如何在同一个请求内传递数据？

对象 [`*macaron.Context`](https://gowalker.org/github.com/Unknwon/macaron#Context) 中包含一个类型为 `map[string]interface{}` 的字段 `Data` 可供您在同个请求的不同处理器之间传递数据。

可以到 [这里](../middlewares/routing#%E9%AB%98%E7%BA%A7%E8%B7%AF%E7%94%B1%E5%AE%9A%E4%B9%89) 查看使用方法。


### 为什么不直接使用 Martini 而要另外创建一个框架？

- 集成常用组件和方法来减少反射次数。
- 使用速度更快的多叉树路由替换原本的路由层。
- 更好地驱动 [Gogs](http://gogs.io) 项目。
- 对 Martini 源码进行一次深度学习。

### 为什么 Logo 是一条龙？

不应该是一种甜品吗？

正所谓 `马卡龙`，此龙乃是名为 `马卡` 的龙，哈哈！

### 有代码实时编译运行工具吗？

[Bra](https://github.com/Unknwon/bra) 可以作为 Macaron 应用的实时编译运行工具。