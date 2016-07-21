# Recovery

[This is a middleware](https://github.com/iris-contrib/middleware/tree/master/recovery).


Safety recover the server from panic.

```
recovery.New(...*logger.Logger)
```

```go
package main
import (
"github.com/kataras/iris"
"github.com/iris-contrib/middleware/recovery"
)
func main() {
iris.Use(recovery.New(iris.Logger)) // optional parameter is the logger which the stack of the panic will be printed, here we're using the default station's Logger
iris.Get("/", func(ctx *iris.Context) {
ctx.Write("Hi, let's panic")
panic("errorrrrrrrrrrrrrrr")
})
iris.Listen(":8080")
}
?1��]
