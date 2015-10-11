---
name: Middlewares
---

# Middlewares and helper modules

Middlewares and helper modules allow you easily plugin/unplugin features for your Macaron applications.

There are already many [middlewares and modules](https://github.com/go-macaron) to simplify your work:

- [gzip](/docs/middlewares/gzip) - Gzip compression to all responses
- [binding](/docs/middlewares/binding) - Request data binding and validation
- [i18n](/docs/middlewares/i18n) - Internationalization and Localization
- [cache](/docs/middlewares/cache) - Cache manager
- [session](/docs/middlewares/session) - Session manager
- [csrf](/docs/middlewares/csrf) - Generates and validates CSRF tokens
- [captcha](/docs/middlewares/captcha) - Captcha service
- [pongo2](https://github.com/macaron-contrib/pongo2) - Pongo2 template engine support
- [sockets](https://github.com/macaron-contrib/sockets) - WebSockets channels binding
- [bindata](/docs/middlewares/bindata) - Embed binary data as static and template files
- [toolbox](https://github.com/macaron-contrib/toolbox) - Health check, pprof, profile and statistic services
- [oauth2](https://github.com/macaron-contrib/oauth2) - OAuth 2.0 backend client
- [switcher](/docs/middlewares/switcher) - Multiple-site support
- [method](https://github.com/macaron-contrib/method) - HTTP method override
- [permissions2](https://github.com/xyproto/permissions2) - Cookies, users and permissions
- [renders](https://github.com/macaron-contrib/renders) - Beego-like render engine(Macaron has built-in template engine, this is another option)

### Best register order for middlewares

Some middlewares depends on others, here is a list for best ordering:

1. `macaron.Logger()`
2. `macaron.Recovery()`
3. `macaron.Gziper()`
4. `macaron.Static()`
5. `macaron.Renderer()`/`pongo2.Pongoer()`
6. `i18n.I18n()`
7. `cache.Cacher()`
8. `captcha.Captchaer()`
9. `session.Sessioner()`
10. `csrf.Csrfer()`
11. `toolbox.Toolboxer()`
