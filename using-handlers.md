# Using Handlers

```go

type myHandlerGet struct {
}

func (m myHandlerGet) Serve(c *iris.Context) {
    c.Write("From %s", c.PathString())
}

//and so on


iris.Handle("GET", "/get", myHandlerGet{})
iris.Handle("POST", "/post", post)
iris.Handle("PUT", "/put", put)
iris.Handle("DELETE", "/delete", del)
```