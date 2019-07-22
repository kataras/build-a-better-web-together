# Handle HTTP errors

You can define your own handlers when a specific http error code occurs.

> Error codes are the http status codes that are bigger than or equal to 400, like 404 not found and 500 internal server.

Example code:

```go
package main

import "github.com/kataras/iris"

func main(){
    app := iris.New()
    app.OnErrorCode(iris.StatusNotFound, notFound)
    app.OnErrorCode(iris.StatusInternalServerError, internalServerError)
    // to register a handler for all "error"
    // status codes(kataras/iris/context.StatusCodeNotSuccessful)
    // defaults to < 200 || >= 400:
    // app.OnAnyErrorCode(handler)
    app.Get("/", index)
    app.Run(iris.Addr(":8080"))
}

func notFound(ctx iris.Context) {
    // when 404 then render the template
    // $views_dir/errors/404.html
    ctx.View("errors/404.html")
}

func internalServerError(ctx iris.Context) {
    ctx.WriteString("Oups something went wrong, try again")
}

func index(ctx context.Context) {
    ctx.View("index.html")
}
```

