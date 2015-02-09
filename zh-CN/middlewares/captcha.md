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

想要使用该中间件，您必须同时使用 [cache](cache) 中间件。

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

## 自定义选项

该服务允许接受一个参数来进行自定义选项（[`captcha.Options`](https://gowalker.org/github.com/macaron-contrib/captcha#Options)）：

```go
// ...
m.Use(captcha.Captchaer(captcha.Options{
    // 获取验证码图片的 URL 前缀，默认为 "/captcha/"
    URLPrefix:			"/captcha/",
    // 表单隐藏元素的 ID 名称，默认为 "captcha_id"
    FieldIdName:		"captcha_id",
    // 用户输入验证码值的元素 ID，默认为 "captcha"
    FieldCaptchaName:	"captcha",
    // 验证字符的个数，默认为 6
    ChallengeNums:		6,
    // 验证码图片的宽度，默认为 240 像素
    Width:				240,
    // 验证码图片的高度，默认为 80 像素
    Height:				80,
    // 验证码过期时间，默认为 600 秒
    Expiration:			600,
    // 用于存储验证码正确值的 Cache 键名，默认为 "captcha_"
    CachePrefix:		"captcha_",
}))
// ...
```
