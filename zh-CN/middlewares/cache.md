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


目前有 8 款内置的适配器，除了 **内存** 和 **文件** 提供器外，您都必须显式导入其它适配器的驱动。

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
    Adapter:       "file",
    AdapterConfig: "data/caches",
}))
//...
```

### Redis

**特别注意** 只能存取 string 和 int 相关类型。

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

当您使用 Redis 作为缓存器时，可以通过将 `OccupyMode` 的值设置为 `true` 来启用独占模式。在该模式下，缓存器将直接占用所选用的整个数据库，而不是通过维护一个索引集合来判断哪些数据是属于您的应用的。当您的缓存数据非常巨大时，该模式可以有效降低应用的 CPU 和内存使用率。

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

可以使用以下 SQL 语句创建数据库：

```sql
CREATE TABLE session (
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

可以使用以下 SQL 语句创建数据库：

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
