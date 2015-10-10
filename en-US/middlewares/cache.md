---
root: false
name: Cache
sort: 6
---

# Cache

Middleware cache provides cache management for Macaron [Instances](../intro/core_concepts#instances).

- [GitHub](https://github.com/macaron-contrib/cache)
- [API Reference](https://gowalker.org/github.com/macaron-contrib/cache)

## Installation

    go get github.com/macaron-contrib/cache

## Usage

```go
import (
    "github.com/Unknwon/macaron"
    "github.com/macaron-contrib/cache"
)

func main() {
    m := macaron.Classic()
    m.Use(cache.Cacher())

    m.Get("/", func(c cache.Cache) string {
        c.Put("cache", "cache middleware", 120)
        return c.Get("cache")
    })

    m.Run()
}
```

## Options

`cache.Cacher` comes with a variety of configuration options([`cache.Options`](https://gowalker.org/github.com/macaron-contrib/cache#Options)):

```go
//...
m.Use(cache.Cacher(cache.Options{
    // Name of adapter. Default is "memory".
    Adapter:        "memory",
    // Adapter configuration, it's corresponding to adapter.
    AdapterConfig:  "",
    // GC interval time in seconds. Default is 60.
    Interval:       60,
    // Configuration section name. Default is "cache".
    Section:        "cache",
    }))
//...
```

## Adapters

There are 8 built-in implementations of cache adapter, you have to import adapter driver explicitly except for **memory** and **file** adapters.

Following are some basic usage examples for adapters.

### Memory

```go
//...
m.Use(cache.Cacher())
//...
```

### File

```go
//...
m.Use(cache.Cacher(cache.Options{
    Adapter:       "file",
    AdapterConfig: "data/caches",
}))
//...
```

### Redis

**Notice** Only string and int-type are allowed.

```go
import _ "github.com/macaron-contrib/cache/redis"

//...
m.Use(cache.Cacher(cache.Options{
    Adapter:       "redis",
    // e.g.: network=tcp,addr=127.0.0.1:6379,password=macaron,db=0,pool_size=100,idle_timeout=180,hset_name=MacaronCache,prefix=cache:
    AdapterConfig: "addr=127.0.0.1:6379,password=macaron",
    OccupyMode:    false,
}))
//...
```

There is a special **occupy mode** for Redis cacher when you want to use entire database selection with large amount of cache data. By setting `OccupyMode` to `true` to enable this mode, then cacher will stop maintaining the index collection of cache data that is used to determine what data are belonging to your app, and helps you reduce CPU and memory usage in such cases.

### Memcache

```go
import _ "github.com/macaron-contrib/cache/memcache"

//...
m.Use(cache.Cacher(cache.Options{
    Adapter:       "memcache",
    // e.g.: 127.0.0.1:9090;127.0.0.1:9091
    AdapterConfig: "127.0.0.1:11211",
}))
//...
```

### PostgreSQL

Use following SQL to create database:

```sql
CREATE TABLE cache (
    key       CHAR(32) NOT NULL,
    data      BYTEA,
    created   INTEGER NOT NULL,
    expire    INTEGER NOT NULL,
    PRIMARY KEY (key)
);
```

```go
import _ "github.com/macaron-contrib/cache/postgres"

//...
m.Use(cache.Cacher(cache.Options{
    Adapter:       "postgres",
    AdapterConfig: "user=a password=b host=localhost port=5432 dbname=c sslmode=disable",
}))
//...
```

### MySQL

Use following SQL to create database:

```sql
CREATE TABLE `cache` (
    `key`       CHAR(32) NOT NULL,
    `data`      BLOB,
    `created`   INT(11) UNSIGNED NOT NULL,
    `expire`    INT(11) UNSIGNED NOT NULL,
    PRIMARY KEY (`key`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;
```

```go
import _ "github.com/macaron-contrib/cache/mysql"

//...
m.Use(cache.Cacher(cache.Options{
    Adapter:       "mysql",
    AdapterConfig: "username:password@protocol(address)/dbname?param=value",
}))
//...
```

### Ledis

```go
import _ "github.com/macaron-contrib/cache/ledis"

//...
m.Use(cache.Cacher(cache.Options{
    Adapter:       "ledis",
    AdapterConfig: "data_dir=./app.db,db=0",
}))
//...
```

### Nodb

```go
import _ "github.com/macaron-contrib/cache/nodb"

//...
m.Use(cache.Cacher(cache.Options{
    Adapter:       "nodb",
    AdapterConfig: "data/cache.db",
}))
//...
```
