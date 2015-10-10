---
root: false
name: 会话管理（Session）
sort: 7
---

# 会话管理（Session）

中间件 session 为 [Macaron 实例](../intro/core_concepts#macaron-%E5%AE%9E%E4%BE%8B) 提供了会话管理的功能。

- [GitHub](https://github.com/macaron-contrib/session)
- [API 文档](https://gowalker.org/github.com/macaron-contrib/session)

## 下载安装

    go get github.com/macaron-contrib/session

## 使用示例

```go
import (
    "github.com/Unknwon/macaron"
    "github.com/macaron-contrib/session"
)

func main() {
    m := macaron.Classic()
    m.Use(session.Sessioner())

    m.Get("/", func(sess session.Store) string {
        sess.Set("session", "session middleware")
        return sess.Get("session").(string)
    })

    m.Get("/signup", func(ctx *macaron.Context, f *session.Flash) {
        f.Success("yes!!!")
        f.Error("opps...")
        f.Info("aha?!")
        f.Warning("Just be careful.")
        ctx.HTML(200, "signup")
    })

    m.Run()
}
```

```html
<!-- templates/signup.tmpl -->
<h2>{{.Flash.SuccessMsg}}</h2>
<h2>{{.Flash.ErrorMsg}}</h2>
<h2>{{.Flash.InfoMsg}}</h2>
<h2>{{.Flash.WarningMsg}}</h2>
```

### Pongo2

如果您正在使用 [pongo2](https://github.com/macaron-contrib/pongo2) 作为应用的模板引擎，则需要对 HTML 进行如下修改：

```html
<!-- templates/signup.tmpl -->
<h2>{{Flash.SuccessMsg}}</h2>
<h2>{{Flash.ErrorMsg}}</h2>
<h2>{{Flash.InfoMsg}}</h2>
<h2>{{Flash.WarningMsg}}</h2>
```

### 将 Flash 输出到当前响应

默认情况下，Flash 的数据只会在相对应会话的下一个响应中使用，但函数 `Success`、`Error`、`Info` 和 `Warning` 均接受第二个参数来指示是否在当前响应输出数据：

```go
// ...
f.Success("yes!!!", true)
f.Error("opps...", true)
f.Info("aha?!", true)
f.Warning("Just be careful.", true)
// ...
```

但是请注意，不管您选择什么时候输出 Flash 的数据，它都只能够被使用一次。

## 自定义选项

该服务允许接受一个参数来进行自定义选项（[`session.Options`](https://gowalker.org/github.com/macaron-contrib/session#Options)）：

```go
//...
m.Use(session.Sessioner(session.Options{
    // 提供器的名称，默认为 "memory"
    Provider:       "memory",
    // 提供器的配置，根据提供器而不同
    ProviderConfig: "",
    // 用于存放会话 ID 的 Cookie 名称，默认为 "MacaronSession"
    CookieName:     "MacaronSession",
    // Cookie 储存路径，默认为 "/"
    CookiePath:     "/",
    // GC 执行时间间隔，默认为 3600 秒
    Gclifetime:     3600,
    // 最大生存时间，默认和 GC 执行时间间隔相同
    Maxlifetime:    3600,
    // 仅限使用 HTTPS，默认为 false
    Secure:         false,
    // Cookie 生存时间，默认为 0 秒
    CookieLifeTime: 0,
    // Cookie 储存域名，默认为空
    Domain:         "",
    // 会话 ID 长度，默认为 16 位
    IDLength:       16,
    // 配置分区名称，默认为 "session"
    Section:        "session",
}))D
//...
```

## 提供器

目前有 9 款内置的提供器，除了 **内存** 和 **文件** 提供器外，您都必须显式导入其它提供器的驱动。

以下为提供器的基本用法：

### 内存

```go
//...
m.Use(session.Sessioner())
//...
```

### 文件

```go
//...
m.Use(session.Sessioner(session.Options{
    Provider:       "file",
    ProviderConfig: "data/sessions",
}))
//...
```

### Redis

```go
import _ "github.com/macaron-contrib/session/redis"

//...
m.Use(session.Sessioner(session.Options{
    Provider:       "redis",
    // e.g.: network=tcp,addr=127.0.0.1:6379,password=macaron,db=0,pool_size=100,idle_timeout=180,prefix=session:
    ProviderConfig: "addr=127.0.0.1:6379,password=macaron",
}))
//...
```

### Memcache

```go
import _ "github.com/macaron-contrib/session/memcache"

//...
m.Use(session.Sessioner(session.Options{
    Provider:       "memcache",
    // e.g.: 127.0.0.1:9090;127.0.0.1:9091
    ProviderConfig: "127.0.0.1:9090",
}))
//...
```

### PostgreSQL

可以使用以下 SQL 语句创建数据库（请确保 `key` 的长度和您设置的 `Options.IDLength` 一致）：

```sql
CREATE TABLE session (
    key       CHAR(16) NOT NULL,
    data      BYTEA,
    expiry    INTEGER NOT NULL,
    PRIMARY KEY (key)
);
```

```go
import _ "github.com/macaron-contrib/session/postgres"

//...
m.Use(session.Sessioner(session.Options{
    Provider:       "postgres",
    ProviderConfig: "user=a password=b dbname=c sslmode=disable",
}))
//...
```

### MySQL

可以使用以下 SQL 语句创建数据库：

```sql
CREATE TABLE `session` (
    `key`       CHAR(16) NOT NULL,
    `data`      BLOB,
    `expiry`    INT(11) UNSIGNED NOT NULL,
    PRIMARY KEY (`key`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;
```

```go
import _ "github.com/macaron-contrib/session/mysql"

//...
m.Use(session.Sessioner(session.Options{
    Provider:       "mysql",
    ProviderConfig: "username:password@protocol(address)/dbname?param=value",
}))
//...
```

### Couchbase

```go
import _ "github.com/macaron-contrib/session/couchbase"

//...
m.Use(session.Sessioner(session.Options{
    Provider:       "couchbase",
    ProviderConfig: "username:password@protocol(address)/dbname?param=value",
}))
//...
```

### Ledis

```go
import _ "github.com/macaron-contrib/session/ledis"

//...
m.Use(session.Sessioner(session.Options{
    Provider:       "ledis",
    ProviderConfig: "data_dir=./app.db,db=0",
}))
//...
```

### Nodb

```go
import _ "github.com/macaron-contrib/session/nodb"

//...
m.Use(session.Sessioner(session.Options{
    Provider:       "nodb",
    ProviderConfig: "data/cache.db",
}))
//...
```

## 实现提供器接口

如果您需要实现自己的会话存储和提供器，可以通过实现下面两个接口实现，同时还可以将 **内存** 提供器作为学习案例。

```go
// RawStore is the interface that operates the session data.
type RawStore interface {
	// Set sets value to given key in session.
	Set(key, value interface{}) error
	// Get gets value by given key in session.
	Get(key interface{}) interface{}
	// Delete deletes a key from session.
	Delete(key interface{}) error
	// ID returns current session ID.
	ID() string
	// Release releases session resource and save data to provider.
	Release() error
	// Flush deletes all session data.
	Flush() error
}

// Provider is the interface that provides session manipulations.
type Provider interface {
	// Init initializes session provider.
	Init(gclifetime int64, config string) error
	// Read returns raw session store by session ID.
	Read(sid string) (RawStore, error)
	// Exist returns true if session with given ID exists.
	Exist(sid string) bool
	// Destory deletes a session by session ID.
	Destory(sid string) error
	// Regenerate regenerates a session store from old session ID to new one.
	Regenerate(oldsid, sid string) (RawStore, error)
	// Count counts and returns number of sessions.
	Count() int
	// GC calls GC to clean expired sessions.
	GC()
}
```
