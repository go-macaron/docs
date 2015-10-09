---
root: true
name: FAQs
sort: 2
---

# FAQs 

### How do I integrate with existing servers?

Every Macaron [instance](/docs/intro/core_concepts#instances) implements [`http.Handler`](https://gowalker.org/net/http#Handler), so it can easily be used to serve subtrees on existing Go servers. For example this is a working Macaron app for Google App Engine:

```go
package hello

import (
	"net/http"
	
	"github.com/Unknwon/macaron"
)

func init() {
	m := macaron.Classic()
	m.Get("/", func() string {
		return "Hello world!"
	})
	http.Handle("/", m)
}
```

### How do I change the port/host?

Macaron's `Run` function looks for the `PORT` and `HOST` environment variables and uses those. Otherwise Macaron will default to [localhost:4000](http://localhost:4000).
To have more flexibility over port and host, use the [`http.ListenAndServe`](https://gowalker.org/net/http#ListenAndServe) function instead.

```go
m := macaron.Classic()
// ...
log.Fatal(http.ListenAndServe(":8080", m))
```

Or following ways:

- `m.Run("0.0.0.0")`, listen on `0.0.0.0:4000`
- `m.Run(8080)`, listen on `0.0.0.0:8080`
- `m.Run("0.0.0.0", 8080)`, listen on `0.0.0.0:8080`

### How do I pass data in request-level other than service inject?

There is a field called `Data` with type `map[string]interface{}` in [`*macaron.Context`](https://gowalker.org/github.com/Unknwon/macaron#Context) where you can store and retrieve any type of data. It comes with [`*macaron.Context`](https://gowalker.org/github.com/Unknwon/macaron#Context) so every request is independent.

See example [here](../middlewares/routing#advanced-routing).

### What's the idea behind this other than Martini?

- Integrate frequently used middlewares and helper methods with less reflection.
- Replace default router with faster multi-tree router.
- To make much easier power [Gogs](http://gogs.io) project.
- Make a deep source study against Martini.

### Why Logo is a dragon?

Shouldn't it be some sort of dessert?

The transliteration of Macaron in Chinese is `Maca Long`, `Long` means dragon, so actually the Logo is a dragon whose name is `Maca`. Hah!

### Live code reload?

[Bra](https://github.com/Unknwon/bra) is the prefect fit for live reloading Macaron apps.