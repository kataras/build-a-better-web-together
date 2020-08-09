# Wrap the Router

You may never need this one but it's here in case.

There are times that you may need to override or decide whether the Router will be executed on an incoming request. If you've any previous experience with the `net/http` and other web frameworks this function will be familiar with you (it has the form of a net/http middleware, but instead of accepting the next handler it accepts the Router as a function to be executed or not).

```go
// WrapperFunc is used as an expected input parameter signature
// for the WrapRouter. It's a "low-level" signature which is compatible
// with the net/http.
// It's being used to run or no run the router based on a custom logic.
type WrapperFunc func(w http.ResponseWriter, r *http.Request, router http.HandlerFunc)

// WrapRouter adds a wrapper on the top of the main router.
// Usually it's useful for third-party middleware
// when need to wrap the entire application with a middleware like CORS.
//
// Developers can add more than one wrappers,
// those wrappers' execution comes from last to first.
// That means that the second wrapper will wrap the first, and so on.
//
// Before build.
func WrapRouter(wrapperFunc WrapperFunc)
```

The Router searches for its routes based on the `Subdomain`, `HTTP Method` and its dynamic `Path`. A Router Wrapper can override that behavior and execute custom code.

In this example you'll just see one use case of .WrapRouter. You can use the .WrapRouter to add custom logic when or when not the router should be executed in order to execute the registered routes' handlers. This is just for the proof of concept, you can skip this tutorial.

Example Code:

```go
package main

import (
    "net/http"
    "strings"

    "github.com/kataras/iris/v12"
)

func newApp() *iris.Application {
    app := iris.New()

    app.OnErrorCode(iris.StatusNotFound, func(ctx iris.Context) {
        ctx.HTML("<b>Resource Not found</b>")
    })

    app.HandleDir("/", "./public")

    app.Get("/profile/{username}", func(ctx iris.Context) {
        ctx.Writef("Hello %s", ctx.Params().Get("username"))
    })

    myOtherHandler := func(ctx iris.Context) {
        ctx.Writef("inside a handler which is fired manually by our custom router wrapper")
    }

    // wrap the router with a native net/http handler.
    // if url does not contain any "." (i.e: .css, .js...)
    // (depends on the app , you may need to add more file-server exceptions),
    // then the handler will execute the router that is responsible for the
    // registered routes (look "/" and "/profile/{username}")
    // if not then it will serve the files based on the root "/" path.
    app.WrapRouter(func(w http.ResponseWriter, r *http.Request, router http.HandlerFunc) {
        path := r.URL.Path

        if strings.HasPrefix(path, "/other") {
                // acquire and release a context in order to use it to execute
                // our custom handler
                // remember: if you are going to call just a net/http middleware
                // then you DONT have to acquire and release the context here.
                ctx := app.ContextPool.Acquire(w, r)
                myOtherHandler(ctx)
                app.ContextPool.Release(ctx)
                return
            }

            // else continue serving routes as usual.
            router.ServeHTTP(w, r) 
    })

    return app
}

func main() {
    app := newApp()

    // http://localhost:8080
    // http://localhost:8080/index.html
    // http://localhost:8080/app.js
    // http://localhost:8080/css/main.css
    // http://localhost:8080/profile/anyusername
    // http://localhost:8080/other/random
    app.Listen(":8080")

    // Note: In this example we just saw one use case,
    // you may want to .WrapRouter or .Downgrade in order to
    // bypass the Iris' default router, i.e:
    // you can use that method to setup custom proxies too.
}
```

There is not much to say here, it's just a function wrapper which accepts the native response writer and request and the next handler which is the Iris' Router itself, it's being or not executed whether is called or not, **it's a middleware for the whole Router**.

