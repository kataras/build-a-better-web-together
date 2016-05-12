# Recovery

[This is a middleware](https://github.com/kataras/iris/tree/master/middleware/recovery)


Safety recover the server from panic.

```
recovery.New(...io.Writer)
```

```go

package main

import (
    "github.com/kataras/iris"
    "github.com/kataras/iris/middleware/recovery"
    "os"
)

func main() {

    iris.Use(recovery.New(os.Stderr)) // optional

    iris.Get("/", func(ctx *iris.Context) {
        ctx.Write("Hi, let's panic")
        panic("Something bad!")
    })

    iris.Listen(":8080")
}

```