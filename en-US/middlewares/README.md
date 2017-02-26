---
name: Middlewares
---

# Middlewares and helper modules

Middlewares and helper modules allow you easily plugin/unplugin features for your Macaron applications.

There are already many [middlewares and modules](https://github.com/go-macaron) to simplify your work:
- [auth](https://github.com/go-macaron/auth) - HTTP Basic authentication
- [bindata](/docs/middlewares/bindata) - Embed binary data as static and template files
- [binding](/docs/middlewares/binding) - Request data binding and validation
- [cache](/docs/middlewares/cache) - Cache manager
- [captcha](/docs/middlewares/captcha) - Captcha service
- [csrf](/docs/middlewares/csrf) - Generates and validates CSRF tokens
- [gzip](/docs/middlewares/gzip) - Gzip compression to all responses
- [i18n](/docs/middlewares/i18n) - Internationalization and Localization
- [inject](https://github.com/go-macaron/inject) - Map and inject dependencies
- [jade](https://github.com/go-macaron/jade) - Jade templating engine
- [method](https://github.com/go-macaron/method) - HTTP method override
- [oauth2](https://github.com/go-macaron/oauth2) - OAuth 2.0 backend client
- [permissions2](https://github.com/xyproto/permissions2) - Cookies, users and permissions
- [pongo2](https://github.com/go-macaron/pongo2) - Pongo2 template engine support
- [renders](https://github.com/go-macaron/renders) - Beego-like render engine(Macaron has built-in template engine, this is another option)
- [session](/docs/middlewares/session) - Session manager
- [sockets](https://github.com/go-macaron/sockets) - WebSockets channels binding
- [switcher](/docs/middlewares/switcher) - Multiple-site support
- [toolbox](https://github.com/go-macaron/toolbox) - Health check, pprof, profile and statistic services

### Best register order for middlewares

Some middlewares depends on others, here is a list for best ordering:

1. `macaron.Logger()`
2. `macaron.Recovery()`
3. `gzip.Gziper()`
4. `macaron.Static()`
5. `macaron.Renderer()`/`pongo2.Pongoer()`
6. `i18n.I18n()`
7. `cache.Cacher()`
8. `captcha.Captchaer()`
9. `session.Sessioner()`
10. `csrf.Csrfer()`
11. `toolbox.Toolboxer()`
