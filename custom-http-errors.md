# Custom HTTP Errors

You can define your own handlers when http error occurs.

```go
package main

import (
	"github.com/kataras/iris"
)

func main() {

	iris.OnError(iris.StatusInternalServerError, func(ctx *iris.Context) {
		ctx.Write(iris.StatusText(iris.StatusInternalServerError)) // Outputs: Not Found
		ctx.SetStatusCode(iris.StatusInternalServerError)          // 404
		iris.Logger().Printf("http status: 500 happened!")
	})

	iris.OnError(iris.StatusNotFound, func(ctx *iris.Context) {
		ctx.Write(iris.StatusText(iris.StatusNotFound)) // Outputs: Internal Server Error
		ctx.SetStatusCode(iris.StatusNotFound)          // 500

		iris.Logger().Printf("http status: 404 happened!")
	})

	// emit the errors to test them
	iris.Get("/500", func(ctx *iris.Context) {
		ctx.EmitError(iris.StatusInternalServerError) // ctx.Panic()
	})

	iris.Get("/404", func(ctx *iris.Context) {
		ctx.EmitError(iris.StatusNotFound) // ctx.NotFound()
	})

	println("Server is running at: 80")
	iris.Listen(":80")

}


```