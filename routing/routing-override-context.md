# Override Context

In this section you will learn how to override the existing [Context](https://godoc.org/github.com/kataras/iris/context#Context)'s methods.

The [Context](https://godoc.org/github.com/kataras/iris/context#Context) is an interface. However as you probably know, when using other frameworks you don't have that functionality of overriding even if it's used as an interface. With Iris you can **attach** your implementation to the **context pool** itself using the `app.ContextPool.Attach` method.

1. Let's get started by importing the `"github.com/kataras/iris/context"` which is required here.
2. Secondly, create your own Context implementation.
3. Add the `Do` and `Next` methods where they just call the `context.Do` and `context.Next` package-level functions.
4. Use the Application's `ContextPool` to set it as the Context implementation that should be used for the route's handlers.

Example Code:

Please read the _comments_ too.

```go
package main

import (
    "reflect"

    "github.com/kataras/iris"
    // 1.
    "github.com/kataras/iris/context"
)

// 2.
// Create your own custom Context, put any fields you'll need.
type MyContext struct {
    // Embed the `context.Context` - 
    // It's totally optional but you will need this if you
    // don't want to override all the context's methods!
    context.Context
}

// Optionally: validate MyContext implements context.Context on compile-time.
var _ context.Context = &MyContext{}

// 3.
func (ctx *MyContext) Do(handlers context.Handlers) {
    context.Do(ctx, handlers)
}


// 3.
func (ctx *MyContext) Next() {
    context.Next(ctx)
}

// [Override any context's method you want here...]
// Like the HTML below:

func (ctx *MyContext) HTML(htmlContents string) (int, error) {
    ctx.Application().Logger().Infof("Executing .HTML function from MyContext")

    ctx.ContentType("text/html")
    return ctx.WriteString(htmlContents)
}

func main() {
    app := iris.New()

    // 4.
    app.ContextPool.Attach(func() context.Context {
        return &MyContext{
            // If you use the embedded Context,
            // call the `context.NewContext` to create one:
            Context: context.NewContext(app),
        }
    })

    // Register a view engine on .html files inside the ./view/** directory.
    app.RegisterView(iris.HTML("./view", ".html"))

    // Register your route, as you normally do
    app.Handle("GET", "/", recordWhichContextForExample,
    func(ctx context.Context) {
        // use the context's overridden HTML method.
        ctx.HTML("<h1> Hello from my custom context's HTML! </h1>")
    })

    // This will be executed by the
    // MyContext.Context embedded default context
    // when MyContext is not directly define the View function by itself.
    app.Handle("GET", "/hi/{firstname:alphabetical}",recordWhichContextForExample,
    func(ctx context.Context) {
        firstname := ctx.Values().GetString("firstname")

        ctx.ViewData("firstname", firstname)
        ctx.Gzip(true)

        ctx.View("hi.html")
    })

    app.Run(iris.Addr(":8080"))
}

// Should always print "($PATH) Handler is executing from 'MyContext'"
func recordWhichContextForExample(ctx context.Context) {
    ctx.Application().Logger().Infof("(%s) Handler is executing from: '%s'",
        ctx.Path(), reflect.TypeOf(ctx).Elem().Name())

    ctx.Next()
}
```

