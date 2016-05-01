# Using HandlerFuncs

HandlerFuncs should implement the Serve(*Context) func.
HandlerFunc is most simple method to register a route or a middleware, but under the hoods it's acts like a Handler. It's implements the Handler interface as well:

```go
type HandlerFunc func(*Context)

func (h HandlerFunc) Serve(c *Context) {
	h(c)
}

```
HandlerFuncs shoud have this function signature:
```go
func handlerFunc(c *iris.Context)  {
	c.Write("Hello")
}


iris.HandleFunc("GET","/letsgetit",handlerFunc)
//OR
iris.Get("/get", handlerFunc)
iris.Post("/post", handlerFunc)
iris.Put("/put", handlerFunc)
iris.Delete("/delete", handlerFunc)
```