---
name: 中间件模块
---

# 中间件及辅助模块

中间件及辅助模块允许您轻易地对模块的进行接入与解除到您的 Macaron 应用中。

现在已经有许多 [中间件和模块](https://github.com/go-macaron) 来简化您的工作：

- [gzip](/docs/middlewares/gzip) - Gzip 压缩所有响应
- [binding](/docs/middlewares/binding) - 请求数据绑定和校验
- [i18n](/docs/middlewares/i18n) - 应用的国际化与本地化
- [cache](/docs/middlewares/cache) - Cache 管理器
- [session](/docs/middlewares/session) - Session 管理器
- [csrf](/docs/middlewares/csrf) - 生成和管理 CSRF 令牌
- [captcha](/docs/middlewares/captcha) - 验证码服务
- [pongo2](https://github.com/go-macaron/pongo2) - Pongo2 模板引擎支持
- [sockets](https://github.com/go-macaron/sockets) - WebSockets 管道绑定
- [bindata](/docs/middlewares/bindata) - 嵌入二进制数据作为静态资源和模板文件
- [toolbox](https://github.com/go-macaron/toolbox) - 健康检查、性能调试和路由统计等服务
- [oauth2](https://github.com/go-macaron/oauth2) - OAuth 2.0 服务器后端客户端
- [authz](https://github.com/go-macaron/authz) - 支持ACL, RBAC, ABAC的权限管理，基于[Casbin](https://github.com/casbin/casbin)
- [switcher](/docs/middlewares/switcher) - 多站点支持
- [method](https://github.com/go-macaron/method) - HTTP 方法覆盖
- [permissions2](https://github.com/xyproto/permissions2) - Cookies、多用户和权限管理
- [renders](https://github.com/go-macaron/renders) - 类 Beego 模板引擎（Macaron 已有内置模板引擎，此为可选）

### 注册中间件的最佳顺序

有些中间件会依赖其它中间件，以下为最佳的注册顺序列表：

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
