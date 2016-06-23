# Logger

[This is a middleware](https://github.com/iris-contrib/middleware/tree/master/logger)

Logs the incoming requests

```go
 New(theLogger *logger.Logger, config ...Config) iris.HandlerFunc
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

errorLogger := logger.New(iris.Logger, logger.Config{
		EnableColors: false, //enable it to enable colors for all, disable colors by iris.Logger.ResetColors(), defaults to false
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

iris.Use(logger.New(iris.Logger))
*/
func main() {

	iris.Use(logger.New(iris.Logger))

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
	errorLogger := logger.New(iris.Logger)


	iris.OnError(iris.StatusNotFound, func(ctx *iris.Context) {
		errorLogger.Serve(ctx)
		ctx.Write("My Custom 404 error page ")
	})
	//

	iris.Listen(":8080")

}


```

You can create your **own Logger** to use
```go

import (
    "github.com/kataras/iris/logger"
     mLogger "github.com/iris-contrib/middleware/logger"
)

theLogger := logger.New(config.DefaultLogger())

iris.Use(mLogger.New(theLogger))
```

>Note that: The logger middleware uses the ColorBgOther and ColorFgOther fields.

The configuration struct for the ` iris/logger` is the `iris/config/logger`  

```go
	Logger struct {
		// Out the (file) writer which the messages/logs will printed to
		// Default is os.Stdout
		Out *os.File
		// Prefix the prefix for each message
		// Default is ""
		Prefix string
		// Disabled default is false
		Disabled bool

		// foreground colors single SGR Code

		// ColorFgDefault the foreground color for the normal message bodies
		ColorFgDefault int
		// ColorFgInfo the foreground  color for info messages
		ColorFgInfo int
		// ColorFgSuccess the foreground color for success messages
		ColorFgSuccess int
		// ColorFgWarning the foreground color for warning messages
		ColorFgWarning int
		// ColorFgDanger the foreground color for error messages
		ColorFgDanger int
		// OtherFgColor the foreground color for the rest of the message types
		ColorFgOther int

		// background colors single SGR Code

		// ColorBgDefault the background color for the normal messages
		ColorBgDefault int
		// ColorBgInfo the background  color for info messages
		ColorBgInfo int
		// ColorBgSuccess the background color for success messages
		ColorBgSuccess int
		// ColorBgWarning the background color for warning messages
		ColorBgWarning int
		// ColorBgDanger the background color for error messages
		ColorBgDanger int
		// OtherFgColor the background color for the rest of the message types
		ColorBgOther int

		// banners are the force printed/written messages, doesn't care about Disabled field

		// ColorFgBanner the foreground color for the banner
		ColorFgBanner int
	}


```


The ` config.DefaultLogger()`  returns ` config.Logger` : 

```go
	return Logger{
		Out:      os.Stdout,
		Prefix:   "",
		Disabled: false,
		// foreground colors
		ColorFgDefault: int(color.FgHiWhite),
		ColorFgInfo:    int(color.FgHiCyan),
		ColorFgSuccess: int(color.FgHiGreen),
		ColorFgWarning: int(color.FgHiMagenta),
		ColorFgDanger:  int(color.FgHiRed),
		ColorFgOther:   int(color.FgHiYellow),
		// background colors
		ColorBgDefault: 0,
		ColorBgInfo:    0,
		ColorBgSuccess: 0,
		ColorBgWarning: 0,
		ColorBgDanger:  0,
		ColorBgOther:   0,
		// banner color
		ColorFgBanner: int(color.FgHiBlue),
	}


```
