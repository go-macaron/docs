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

There are 5 built-in implementations of cache adapter, you have to import adapter driver explicitly except for **memory** and **file** adapters.

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
    Provider:       "file",
    ProviderConfig: "data/caches",
}))
//...
```

### Redis

```go
import _ "github.com/macaron-contrib/cache/redis"

//...
m.Use(cache.Cacher(cache.Options{
    Provider:       "redis",
    ProviderConfig: ":6039",
}))
//...
```

### Memcache

```go
import _ "github.com/macaron-contrib/cache/memcache"

//...
m.Use(cache.Cacher(cache.Options{
    Provider:       "memcache",
    ProviderConfig: "127.0.0.1:11211",
}))
//...
```

### Nodb

```go
import _ "github.com/macaron-contrib/cache/nodb"

//...
m.Use(cache.Cacher(cache.Options{
    Provider:       "nodb",
    ProviderConfig: "data/cache.db",
}))
//...
```
