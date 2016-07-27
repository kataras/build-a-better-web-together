# Handlers

Handlers, as the name implies, handle requests. Each of the handler registration methods described in the following subchapters returns a [`RouteNameFunc`](https://godoc.org/github.com/kataras/iris#RouteNameFunc) type.

Handlers must implement the Handler interface:

```go
type Handler interface {
	Serve(*Context)
}
```

Once the handler is registered, we can use the returned [`RouteNameFunc`](https://godoc.org/github.com/kataras/iris#RouteNameFunc) type (which is actually a `func` type) to give name to the this handler registration for easier lookup in code or in templates. For more information, checkout the [Routing and reverse lookups](routing.md) section.
