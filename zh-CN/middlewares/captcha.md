---
root: false
name: 验证码服务
sort: 9
---

# 验证码服务

中间件 captcha 用于为 [Macaron 实例](../intro/core_concepts#macaron-%E5%AE%9E%E4%BE%8B) 提供验证码服务。

- [GitHub](https://github.com/macaron-contrib/captcha)
- [API 文档](https://gowalker.org/github.com/macaron-contrib/captcha)

### 下载安装

    go get github.com/macaron-contrib/captcha

## 使用示例

想要使用该中间件，您必须同时使用 [cache](./cache) 中间件。

```go
// main.go
import (
    "github.com/Unknwon/macaron"
    "github.com/macaron-contrib/cache"
    "github.com/macaron-contrib/captcha"
)

func main() {
    m := macaron.Classic()
    m.Use(cache.Cacher())
    m.Use(captcha.Captchaer())

    m.Get("/", func(ctx *macaron.Context, cpt *captcha.Captcha) string {
        if cpt.VerifyReq(ctx.Req) {
            return "valid captcha"
        }
        return "invalid captcha"
    })

    m.Run()
}
```

```html
<!-- templates/hello.tmpl -->
{{.Captcha.CreateHtml}}
```

## Options

`captcha.Captchaer` comes with a variety of configuration options:

```go
// ...
m.Use(captcha.Captchaer(captcha.Options{
    // URL prefix of getting captcha pictures. Default is "/captcha/".
    URLPrefix:			"/captcha/",
    // Hidden input element ID. Default is "captcha_id".
    FieldIdName:		"captcha_id",
    // User input value element name in request form. Default is "captcha".
    FieldCaptchaName:	"captcha",
    // Challenge number. Default is 6.
    ChallengeNums:		6,
    // Captcha image width. Default is 240.
    Width:				240,
    // Captcha image height. Default is 80.
    Height:				80,
    // Captcha expiration time in seconds. Default is 600.
    Expiration:			600,
    // Cache key prefix captcha characters. Default is "captcha_".
    CachePrefix:		"captcha_",
}))
// ...
```
