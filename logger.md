# Logger

[This is a middleware](https://github.com/iris-contrib/middleware/tree/master/logger)

Logs the incoming requests

```go
 New(config ...Config) iris.HandlerFunc
```

How to use

```go
package main

import (
	"github.com/kataras/iris"
	"github.com/iris-contrib/middleware/logger"
)

/*
With configs:

errorLogger := logger.New(logger.Config{
	// Status displays status code
	Status: true,
	// IP displays request's remote address
	IP: true,
	// Method displays the http method
	Method: true,
	// Path displays the request path
	Path: true,
})

iris.Use(errorLogger)

With default configs:

iris.Use(logger.New())
*/
func main() {

	iris.Use(logger.New())

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Write("hello")
	})

	iris.Get("/1", func(ctx *iris.Context) {
		ctx.Write("hello")
	})

	iris.Get("/2", func(ctx *iris.Context) {
		ctx.Write("hello")
	})

	// log http errors
	errorLogger := logger.New()


	iris.OnError(iris.StatusNotFound, func(ctx *iris.Context) {
		errorLogger.Serve(ctx)
		ctx.Write("My Custom 404 error page ")
	})
	//

	iris.Listen(":8080")

}
