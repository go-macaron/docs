---
root: false
name: 跨域攻击（CSRF）
sort: 8
---

# 跨域请求攻击（CSRF）

中间件 csrf 用于为 [Macaron 实例](../intro/core_concepts#macaron-%E5%AE%9E%E4%BE%8B) 生成和验证 CSRF 令牌。

- [GitHub](https://github.com/macaron-contrib/csrf)
- [API 文档](https://gowalker.org/github.com/macaron-contrib/csrf)

## 下载安装

    go get github.com/macaron-contrib/csrf

## 使用示例

想要使用该中间件，您必须同时使用 [session](session) 中间件。

```go
package main

import (
    "github.com/Unknwon/macaron"
    "github.com/macaron-contrib/csrf"
    "github.com/macaron-contrib/session"
)

func main() {
    m := macaron.Classic()
    m.Use(macaron.Renderer())
    m.Use(session.Sessioner())
    m.Use(csrf.Csrfer())

    // 模拟验证过程，判断 session 中是否存在 uid 数据。
    // 若不存在，则跳转到一个生成 CSRF 的页面。
    m.Get("/", func(ctx *macaron.Context, sess session.Store) {
        if sess.Get("uid") == nil {
            ctx.Redirect("/login")
            return
        }
        ctx.Redirect("/protected")
    })

    // 设置 session 中的 uid 数据。
    m.Get("/login", func(ctx *macaron.Context, sess session.Store) {
        sess.Set("uid", 123456)
        ctx.Redirect("/")
    })

    // 渲染一个需要验证的表单，并传递 CSRF 令牌到表单中。
    m.Get("/protected", func(ctx *macaron.Context, sess session.Store, x csrf.CSRF) {
        if sess.Get("uid") == nil {
            ctx.Redirect("/login", 401)
            return
        }

        ctx.Data["csrf_token"] = x.GetToken()
        ctx.HTML(200, "protected")
    })

    // 验证 CSRF 令牌。
    m.Post("/protected", csrf.Validate, func(ctx *macaron.Context, sess session.Store) {
        if sess.Get("uid") != nil {
            ctx.RenderData(200, []byte("You submitted a valid token"))
            return
        }
        ctx.Redirect("/login", 401)
    })

    m.Run()
}
```

```html
<!-- templates/protected.tmpl -->
<form action="/protected" method="post">
    <input type="hidden" name="_csrf" value="{{.csrf_token}}">
    <button>提交</button>
</form>
```

## 自定义选项

该服务允许接受一个参数来进行自定义选项（[`csrf.Options`](https://gowalker.org/github.com/macaron-contrib/csrf#Options)）：

```go
// ...
m.Use(csrf.Csrfer(csrf.Options{
    // 用于生成令牌的全局秘钥，默认为随机字符串
    Secret:	    "mysecret",
    // 用于传递令牌的 HTTP 请求头信息字段，默认为 "X-CSRFToken"
    Header:		"X-CSRFToken",
    // 用于传递令牌的表单字段名，默认为 "_csrf"
    Form:		"_csrf",
    // 用于传递令牌的 Cookie 名称，默认为 "_csrf"
    Cookie:		"_csrf",
    // Cookie 设置路径，默认为 "/"
    CookiePath:	"/",
    // 用于保存用户 ID 的 session 名称，默认为 "uid"
    SessionKey:	"uid",
    // 用于指定是否将令牌设置到响应的头信息中，默认为 false
    SetHeader:	false,
    // 用于指定是否将令牌设置到响应的 Cookie 中，默认为 false
    SetCookie:  false,
    // 用于指定是否要求只有使用 HTTPS 时才设置 Cookie，默认为 false
    Secure:     false,
    // 用于禁止请求头信息中包括 Origin 字段，默认为 false
    Origin:     false,
    // 错误处理函数，默认为简单的错误输出
    ErrorFunc:  func(w http.ResponseWriter) {
        http.Error(w, "Invalid csrf token.", http.StatusBadRequest)
    },
    }))
// ...
```
