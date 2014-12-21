---
root: false
name: Cross-Site Request Forgery
sort: 8
---

# Cross-Site Request Forgery

Middleware csrf generates and validates CSRF tokens for Macaron [Instances](../intro/core_concepts#instances).

- [GitHub](https://github.com/macaron-contrib/csrf)
- [API Reference](https://gowalker.org/github.com/macaron-contrib/csrf)

## Installation

    go get github.com/macaron-contrib/csrf

## Usage

To use this middleware, you have to enable [session](session) as well.

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

    // Simulate the authentication of a session.
    // If uid exists redirect to a form that requires CSRF protection.
    m.Get("/", func(ctx *macaron.Context, sess session.Store) {
        if sess.Get("uid") == nil {
            ctx.Redirect("/login")
            return
        }
        ctx.Redirect("/protected")
    })

    // Set uid for the session.
    m.Get("/login", func(ctx *macaron.Context, sess session.Store) {
        sess.Set("uid", 123456)
        ctx.Redirect("/")
    })

    // Render a protected form. Passing a csrf token by calling x.GetToken()
    m.Get("/protected", func(ctx *macaron.Context, sess session.Store, x csrf.CSRF) {
        if sess.Get("uid") == nil {
            ctx.Redirect("/login", 401)
            return
        }

        // Pass token to the protected template.
        ctx.Data["csrf_token"] = x.GetToken()
        ctx.HTML(200, "protected")
    })

    // Apply CSRF validation to route.
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
    <button>Submit</button>
</form>
```

## Options

`csrf.Csrfer` comes with a variety of configuration options:

```go
// ...
m.Use(csrf.Csrfer(csrf.Options{
    // The global secret value used to generate Tokens. Default is a random string.
    Secret:	    "mysecret",
    // HTTP header used to set and get token. Default is "X-CSRFToken".
    Header:		"X-CSRFToken",
    // Form value used to set and get token. Default is "_csrf".
    Form:		"_csrf",
    // Cookie value used to set and get token. Default is "_csrf".
    Cookie:		"_csrf",
    // Cookie path. Default is "/".
    CookiePath:	"/",
    // Key used for getting the unique ID per user. Default is "uid".
    SessionKey:	"uid",
    // If true, send token via header. Default is false.
    SetHeader:	false,
    // If true, send token via cookie. Default is false.
    SetCookie:  false,
    // Set the Secure flag to true on the cookie. Default is false.
    Secure:     false,
    // Disallow Origin appear in request header. Default is false.
    Origin:     false,
    // The function called when Validate fails. Default is a simple error print.
    ErrorFunc:  func(w http.ResponseWriter) {
        http.Error(w, "Invalid csrf token.", http.StatusBadRequest)
    },
    }))
// ...
```
