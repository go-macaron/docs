# 初学者指南

在我们开始之前，必须明确的一点就是，文档不会教授您任何有关 Go 语言的基础知识。所有对 Macaron 使用的讲解均是基于您已有的知识基础上展开的。

通过执行以下命令来安装 Macaron：

```sh
go get gopkg.in/macaron.v1
```

并且可以在今后使用以下命令来升级 Macaron：

```sh
go get -u gopkg.in/macaron.v1
```

## 最简示例

创建一个名为 `main.go` 的文件，然后输入以下代码：

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

函数 [`macaron.Classic`](https://gowalker.org/gopkg.in/macaron.v1#Classic) 创建并返回一个 [经典 Macaron](core_concepts.md#jing-dian-macaron) 实例。

方法 [`m.Get`](https://gowalker.org/gopkg.in/macaron.v1#Router_Get) 是用于注册针对 HTTP GET 请求的路由。在本例中，我们注册了针对根路径 `/` 的路由，并提供了一个 [处理器](core_concepts.md#chu-li-qi) 函数来进行简单的处理操作，即返回内容为 `Hello world!` 的字符串作为响应。

您可能会问，为什么处理器函数可以返回一个字符串作为响应？这是由于 [返回值](core_concepts.md#fan-hui-zhi) 所带来的特性。换句话说，我们在本例中使用了 Macaron 中处理器的一个特殊语法来将返回值作为响应内容。

最后，我们调用 [`m.Run`](https://gowalker.org/gopkg.in/macaron.v1#Macaron_Run) 方法来让服务器启动。在默认情况下，[Macaron 实例](core_concepts.md#macaron-shi-li) 会监听 `0.0.0.0:4000`。

接下来，就可以执行命令 `go run main.go` 运行程序。您应该在程序启动后看到一条日志信息：

```sh
[Macaron] listening on 0.0.0.0:4000 (development)
```

现在，打开您的浏览器然后访问 [localhost:4000](http://localhost:4000)。您会发现，一切是如此的美好！

## 扩展示例

现在，让我们对 `main.go` 做出一些修改，以便进行更多的练习。

```go
package main

import (
	"log"
	"net/http"

	"gopkg.in/macaron.v1"
)

func main() {
	m := macaron.Classic()
	m.Get("/", myHandler)

	log.Println("Server is running...")
	log.Println(http.ListenAndServe("0.0.0.0:4000", m))
}

func myHandler(ctx *macaron.Context) string {
	return "the request path is: " + ctx.Req.RequestURI
}
```

当您再次执行命令 `go run main.go` 运行程序的时候，您会看到屏幕上显示的内容为 `the request path is: /`。

那么，是什么改变了事物原本的样貌？（答：爱情）

首先，我们依旧使用了 [经典 Macaron](core_concepts.md#jing-dian-macaron) 来为根路径 `/` 注册针对 HTTP GET 请求的路由。但我们不再使用匿名函数，而是改用名为 `myHandler` 的函数作为处理器。需要注意的是，注册路由时，不需要在函数名称后面加上括号，因为我们不需要在此时调用这个函数。

函数 `myHandler` 接受一个类型为 [`*macaron.Context`](middlewares/core_services.md#qing-qiu-shang-xia-wen-context) 的参数，并返回一个字符串。您可能已经发现我们并没有告诉 Macaron 需要传递什么参数给处理器，而且当您查看 [`m.Get`](https://gowalker.org/gopkg.in/macaron.v1#Router_Get) 方法的声明时会发现，Macaron 实际上将所有的处理器（[`macaron.Handler`](https://gowalker.org/gopkg.in/macaron.v1#Handler)）都当作类型 `interface{}` 来处理。那么，Macaron 又是怎么知道需要传递什么参数来调用处理器并执行逻辑的呢？

这就涉及到 [服务注入](core_concepts.md#fu-wu-zhu-ru) 的概念了， [`*macaron.Context`](middlewares/core_services.md#qing-qiu-shang-xia-wen-context) 就是默认注入的服务之一，所以您可以直接使用它作为参数。如果您不明白怎么注入您自己的服务，没关系，反正还不是时候知道这些。

和之前的例子一样，我们需要让服务器监听在某个地址上。这一次，我们使用 Go 标准库的函数 [`http.ListenAndServe`](https://gowalker.org/net/http#ListenAndServe) 来完成这项操作。如此一来，您便可以发现，任一 [Macaron 实例](core_concepts.md#macaron-shi-li) 都是和标准库完全兼容的。

## 了解更多

您现在已经知道怎么基于 Macaron 来书写简单的代码，请尝试修改上文中的两个示例，并确保您已经完全理解上文中的所有内容。

当您觉得自己已经原地满血复活后，就可以继续学习之后的内容了。