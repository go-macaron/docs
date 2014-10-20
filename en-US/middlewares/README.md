---
root: true
name: Middlewares
sort: 1
---

# Middlewares 

Middlewares allow you easily plugin/unplugin features for your Macaron applications.

There are already many [middlewares](https://github.com/macaron-contrib) to simplify your work:

- [binding](https://github.com/macaron-contrib/binding) - Request data binding and validation
- [i18n](https://github.com/macaron-contrib/i18n) - Internationalization and Localization
- [cache](https://github.com/macaron-contrib/cache) - Cache manager
- [session](https://github.com/macaron-contrib/session) - Session manager
- [csrf](https://github.com/macaron-contrib/csrf) - Generates and validates csrf tokens
- [captcha](https://github.com/macaron-contrib/captcha) - Captcha service
- [pongo2](https://github.com/macaron-contrib/pongo2) - Pongo2 template engine support
- [toolbox](https://github.com/macaron-contrib/toolbox) - Health check, pprof, profile and statistic services
- [switcher](https://github.com/macaron-contrib/switcher) - Multiple-site support
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