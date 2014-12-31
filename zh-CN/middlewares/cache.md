---
root: false
name: 缓存管理（Cache）
sort: 6
---

# 缓存管理（Cache）

中间件 cache 为 [Macaron 实例](../intro/core_concepts#macaron-%E5%AE%9E%E4%BE%8B) 提供了缓存管理的功能。

- [GitHub](https://github.com/macaron-contrib/cache)
- [API Reference](https://gowalker.org/github.com/macaron-contrib/cache)

## 下载安装

    go get github.com/macaron-contrib/cache

## 使用示例

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

## 自定义选项

该服务允许接受一个参数来进行自定义选项（[`cache.Options`](https://gowalker.org/github.com/macaron-contrib/cache#Options)）：

```go
//...
m.Use(cache.Cacher(cache.Options{
    // 适配器的名称，默认为 "memory".
    Adapter:        "memory",
    // 适配器的配置，根据适配器而不同
    AdapterConfig:  "",
    // GC 执行时间间隔，默认为 60 秒
    Interval:       60,
    // 配置分区名称，默认为 "cache"
    Section:        "cache",
    }))
//...
```

## 适配器


目前有 5 款内置的适配器，除了 **内存** 和 **文件** 提供器外，您都必须显式导入其它适配器的驱动。

以下为适配器的基本用法：

### 内存

```go
//...
m.Use(cache.Cacher())
//...
```

### 文件

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
