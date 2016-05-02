# Recovery

[This is a middleware](https://github.com/kataras/iris/tree/development/middleware/recovery)


Safety recover the server from panic.

```go

package main

import (
    "github.com/kataras/iris"
    "github.com/kataras/iris/middleware/recovery"
    "os"
)

func main() {

    iris.Use(recovery.New(os.Stderr)) // optional parameter is the writer which the stack of the panic will be printed

    iris.Get("/", func(ctx *iris.Context) {
        ctx.Write("Hi, let's panic")
        panic("Something bad!")
    })

    iris.Listen(":8080")
}

```