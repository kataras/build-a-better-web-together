# Recovery

[This is a middleware](https://github.com/iris-contrib/middleware/tree/master/recovery).


Safely recover the server from a panic.

```go
recovery.New(...*logger.Logger)
```

```go

package main

import (
	"github.com/kataras/iris"
	"github.com/iris-contrib/middleware/recovery"
)

func main() {

	iris.Use(recovery.Handler)

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Write("Hi, let's panic")
		panic("errorrrrrrrrrrrrrrr")
	})

	iris.Listen(":8080")
}
