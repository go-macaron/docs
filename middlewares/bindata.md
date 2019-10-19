# Embed Binary Data

Package bindata is a helper module that allows to use in-memory static and template files for Macaron [Instances](../core_concepts.md#instances).

- [GitHub](https://github.com/go-macaron/bindata)
- [API Reference](https://gowalker.org/github.com/go-macaron/bindata)

### Installation

```sh
go get github.com/go-macaron/bindata
```

## Usage

Using [go-bindata](https://github.com/go-bindata/go-bindata) convert your template and public directories into individual packages.

Import the packages and use them like the example below.

```go
import (
    "path/to/bindata/public"
    "path/to/bindata/templates"
    "github.com/go-macaron/bindata"
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
