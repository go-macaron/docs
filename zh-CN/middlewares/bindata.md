---
root: false
name: 二进制数据
sort: 12
---

# 嵌入二进制数据

模块 bindata 用于为 [Macaron 实例](../intro/core_concepts#macaron-%E5%AE%9E%E4%BE%8B) 提供支持内存的静态文件服务和模板文件系统。

- [GitHub](https://github.com/macaron-contrib/bindata)
- [API 文档](https://gowalker.org/github.com/macaron-contrib/bindata)

### 下载安装

    go get github.com/macaron-contrib/bindata

## 使用示例

使用 [go-bindata](https://github.com/jteeuwen/go-bindata) 将相应的静态文件和模板文件转换成单独的包。

导入相应的包并通过如下方法实现支持：

```go
import (
    "path/to/bindata/public"
    "path/to/bindata/templates"
    "github.com/macaron-contrib/bindata"
)

m.Use(macaron.Static("public",
    macaron.StaticOptions{
        FileSystem: bindata.Static(bindata.Options{
            Asset:      public.Asset,
            AssetDir:   public.AssetDir,
            AssetNames: public.AssetNames,
            Prefix:     "",
        }),
    },
))

m.Use(macaron.Renderer(macaron.RenderOptions{
    TemplateFileSystem: bindata.Templates(bindata.Options{
        Asset:      templates.Asset,
        AssetDir:   templates.AssetDir,
        AssetNames: templates.AssetNames,
        Prefix:     "",
    }),
}))
```
