---
root: false
name: Multiple Sites
sort: 14
---

# Running Multiple Sites In One App

Module switcher provides host switch functionality for [Macaron](https://github.com/Unknwon/macaron).

- [GitHub](https://github.com/macaron-contrib/switcher)
- [API Reference](https://gowalker.org/github.com/macaron-contrib/switcher)

## Installation

	go get github.com/macaron-contrib/switcher

## Usage

If you want to run 2 instances in one program, Host Switcher is the feature you're looking for.

```go
func main() {
	m1 := macaron.Classic()
	// Register m1 middlewares and routers.

	m2 := macaron.Classic()
	// Register m2 middlewares and routers.

	hs := macaron.NewHostSwitcher()
	// Set instance corresponding to host address.
	hs.Set("gowalker.org", m1)
	hs.Set("gogs.io", m2)
	hs.Run()
}
```

By default, this program will listen on ports `4000`(for `m1`) and `4001`(for `m2`) in `macaron.DEV` mode just for convenience. And only listen on `4000` in `macaron.PROD` mode.

### Dynamic match

In case you have different subdomains that need only one Macaron instance:

```go
// ...
m := macaron.Classic()
// Register m middlewares and routers.

hs := macaron.NewHostSwitcher()
// Set instance corresponding to host address.
hs.Set("*.example.com", m)
hs.Run()
// ...
```

## Others

- See [gogs.io](https://github.com/gogits/gogsweb) as a study example.
