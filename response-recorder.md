# Response Recorder

A response recorder is an Iris specific `http.ResponseWriter` which records the sent body, status code and headers that you can manipulate at any handler inside a route's request handlers chain.

1. Call `Context.Record()` before send data.
2. The `Context.Recorder()` returns a [ResponseRecorder](https://godoc.org/github.com/kataras/iris/context#ResponseRecorder). Its methods can be used to manipulate or retrieve the response.

The `ResponseRecorder` type contains the standard Iris ResponseWriter methods plus the following methods.

Body **returns** the body tracked from the writer so far.
Do not use this for edit.

```go
Body() []byte
```

Use this to clear the body.

```go
ResetBody()
```

Use `Write/Writef/WriteString` to stream write and `SetBody/SetBodyString` to **set** body instead.

```go
Write(contents []byte) (int, error)

Writef(format string, a ...interface{}) (n int, err error)

WriteString(s string) (n int, err error)

SetBody(b []byte)

SetBodyString(s string)
```

Reset headers to their original state, before `Context.Record` call.

```go
ResetHeaders()
```

Clear all headers.

```go
ClearHeaders()
```

Reset resets the response body, headers and the status code header.

```go
Reset()
```

## Example

Record operation log in global Interceptor.

```go
package main

import "github.com/kataras/iris"

func main() {
    app := iris.New()

    // start record.
    app.Use(func(ctx iris.Context) {
        ctx.Record()
        ctx.Next()
    })

    // collect and "log".
    app.Done(func(ctx iris.Context) {
        body := ctx.Recorder().Body()

        // Should print success.
        app.Logger().Infof("sent: %s", string(body))
    })
}
```

Register the routes...

```go
app.Get("/save", func(ctx iris.Context) {
    ctx.WriteString("success")
    ctx.Next() // calls the Done middleware(s).
})
```

Or to remove the need of `ctx.Next` in your main handlers, modify the Iris handlers [execution rules](https://github.com/kataras/iris/blob/master/_examples/mvc/middleware/without-ctx-next/main.go) as follows.

```go
// It applies per Party and its children,
// therefore, you can create a routes := app.Party("/path")
// and set middlewares, their rules and the routes there as well.
app.SetExecutionRules(iris.ExecutionRules{ 
    Done: iris.ExecutionOptions{Force: true},
})

// [The routes...]
app.Get("/save", func(ctx iris.Context) {
    ctx.WriteString("success")
})
```

In addition to that, Iris provides a comprehensive API for **Transactions**. Learn more about it by running an [example](https://github.com/kataras/iris/blob/master/_examples/http_responsewriter/transactions/main.go).
