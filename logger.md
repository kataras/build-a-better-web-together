# Logger


Below you will find a middleware which logs the requests

[This is a middleware](https://github.com/kataras/iris/tree/development/middleware/logger)

```go
Custom(writer io.Writer, prefix string, flag int) iris.HandlerFunc
Default() iris.HandlerFunc
```

Example


```go
package main

import (
    "github.com/kataras/iris"
    "github.com/kataras/iris/middleware/logger"
)

func main() {

    iris.UseFunc(logger.Default())
    // iris.UseFunc(logger.Custom(writer io.Writer, prefix string, flag int))

    iris.Get("/", func(ctx *iris.Context) {
        ctx.Write("hello")
    })

    iris.Get("/1", func(ctx *iris.Context) {
        ctx.Write("hello")
    })

    iris.Get("/3", func(ctx *iris.Context) {
        ctx.Write("hello")
    })

    iris.Listen(":80")

}


```