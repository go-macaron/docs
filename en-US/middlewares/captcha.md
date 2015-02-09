---
root: false
name: Captcha
sort: 9
---

# Captcha

Middleware captcha provides captcha service for Macaron [Instances](../intro/core_concepts#instances).

- [GitHub](https://github.com/macaron-contrib/captcha)
- [API Reference](https://gowalker.org/github.com/macaron-contrib/captcha)

### Installation

    go get github.com/macaron-contrib/captcha

## Usage

To use this middleware, you have to enable [cache](cache) as well.

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
