---
root: false
name: Session
sort: 7
---

# Session

Middleware session provides session management for Macaron [Instances](../intro/core_concepts#instances).

- [GitHub](https://github.com/macaron-contrib/session)
- [API Reference](https://gowalker.org/github.com/macaron-contrib/session)

## Installation

    go get github.com/macaron-contrib/session

## Usage

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

If you're using [pongo2](https://github.com/macaron-contrib/pongo2) as template engine, you will use flash in HTML as follows:

```html
<!-- templates/signup.tmpl -->
<h2>{{Flash.SuccessMsg}}</h2>
<h2>{{Flash.ErrorMsg}}</h2>
<h2>{{Flash.InfoMsg}}</h2>
<h2>{{Flash.WarningMsg}}</h2>
```

### Output flash in current response

By default, flash will be only used for the next coming response corresponding to the session, but functions `Success`, `Error`, `Info` and `Warning` are all accept a second argument to indicate whether output flash in current response or not.

```go
// ...
f.Success("yes!!!", true)
f.Error("opps...", true)
f.Info("aha?!", true)
f.Warning("Just be careful.", true)
// ...
```

But remember, flash can only be used once no matter which way you use.

## Options

`session.Sessioner` comes with a variety of configuration options([`session.Options`](https://gowalker.org/github.com/macaron-contrib/session#Options)):

```go
//...
m.Use(session.Sessioner(session.Options{
    // Name of provider. Default is "memory".
    Provider:       "memory",
    // Provider configuration, it's corresponding to provider.
    ProviderConfig: "",
    // Cookie name to save session ID. Default is "MacaronSession".
    CookieName:     "MacaronSession",
    // Cookie path to store. Default is "/".
    CookiePath:     "/",
    // GC interval time in seconds. Default is 3600.
    Gclifetime:     3600,
    // Max life time in seconds. Default is whatever GC interval time is.
    Maxlifetime:    3600,
    // Use HTTPS only. Default is false.
    Secure:         false,
    // Cookie life time. Default is 0.
    CookieLifeTime: 0,
    // Cookie domain name. Default is empty.
    Domain:         "",
    // Session ID length. Default is 16.
    IDLength:       16,
    // Configuration section name. Default is "session".
    Section:        "session",
}))
//...
```

## Providers

There are 9 built-in implementations of session provider, you have to import provider driver explicitly except for **memory** and **file** providers.

Following are some basic usage examples for providers.

### Memory

```go
//...
m.Use(session.Sessioner())
//...
```

### File

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

Use following SQL to create database(make sure `key` length matches your `Options.IDLength`):

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
    ProviderConfig: "user=a password=b host=localhost port=5432 dbname=c sslmode=disable",
}))
//...
```

### MySQL

Use following SQL to create database:

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

## Implement Provider Interface

In case you need to have your own implementation of session storage and provider, you can implement following two interfaces and take **memory** provider as a study example.

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
