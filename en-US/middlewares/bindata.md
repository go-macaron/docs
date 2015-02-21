---
root: false
name: Binary Data
sort: 12
---

# Embed Binary Data

Package bindata is a helper module that allows to use in-memory static and template files for Macaron [Instances](../intro/core_concepts#instances).

- [GitHub](https://github.com/macaron-contrib/bindata)
- [API Reference](https://gowalker.org/github.com/macaron-contrib/bindata)

### Installation

    go get github.com/macaron-contrib/bindata

## Usage

Using [go-bindata](https://github.com/jteeuwen/go-bindata) convert your template and public directories into individual packages.

Import the packages and use them like the example below.

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
