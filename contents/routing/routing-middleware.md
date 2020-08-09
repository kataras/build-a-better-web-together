# Middleware

When we talk about Middleware in Iris we're talking about running code before and/or after our main handler code in a HTTP request lifecycle. For example, logging middleware might write the incoming request details to a log, then call the handler code, before writing details about the response to the log. One of the cool things about middleware is that these units are extremely flexible and reusable.

A middleware is just a **Handler** form of `func(ctx iris.Context)`, the middleware is being executed when the previous middleware calls the `ctx.Next()`, this can be used for authentication, i.e: if request authenticated then call `ctx.Next()` to process with the rest of the chain of handlers in the request otherwise fire an error response.

## Writing a middleware

```go
package main

import "github.com/kataras/iris/v12"

func main() {
    app := iris.New()
    // or app.Use(before) and app.Done(after).
    app.Get("/", before, mainHandler, after)
    app.Listen(":8080")
}

func before(ctx iris.Context) {
    shareInformation := "this is a sharable information between handlers"

    requestPath := ctx.Path()
    println("Before the mainHandler: " + requestPath)

    ctx.Values().Set("info", shareInformation)
    ctx.Next() // execute the next handler, in this case the main one.
}

func after(ctx iris.Context) {
    println("After the mainHandler")
}

func mainHandler(ctx iris.Context) {
    println("Inside mainHandler")

    // take the info from the "before" handler.
    info := ctx.Values().GetString("info")

    // write something to the client as a response.
    ctx.HTML("<h1>Response</h1>")
    ctx.HTML("<br/> Info: " + info)

    ctx.Next() // execute the "after".
}
```

```bash
$ go run main.go # and navigate to the http://localhost:8080
Now listening on: http://localhost:8080
Application started. Press CTRL+C to shut down.
Before the mainHandler: /
Inside mainHandler
After the mainHandler
```

### Globally

```go
package main

import "github.com/kataras/iris/v12"

func main() {
    app := iris.New()

    // register our routes.
    app.Get("/", indexHandler)
    app.Get("/contact", contactHandler)

    // Order of those calls does not matter,
    // `UseGlobal` and `DoneGlobal` are applied to existing routes
    // and future routes also.
    //
    // Remember: the `Use` and `Done` are applied to the current party's and its children,
    // so if we used the `app.Use/Done before the routes registration
    // it would work like UseGlobal/DoneGlobal in this case,
    // because the `app` is the root "Party".
    app.UseGlobal(before)
    app.DoneGlobal(after)

    app.Listen(":8080")
}

func before(ctx iris.Context) {
     // [...]
}

func after(ctx iris.Context) {
    // [...]
}

func indexHandler(ctx iris.Context) {
    // write something to the client as a response.
    ctx.HTML("<h1>Index</h1>")

    ctx.Next() // execute the "after" handler registered via `Done`.
}

func contactHandler(ctx iris.Context) {
    // write something to the client as a response.
    ctx.HTML("<h1>Contact</h1>")

    ctx.Next() // execute the "after" handler registered via `Done`.
}
```

You could also use the `ExecutionRules` to force Done handlers to be executed without the need of `ctx.Next()` in your route handlers, do it like this:

```go
app.SetExecutionRules(iris.ExecutionRules{
    // Begin: ...
    // Main:  ...
    Done: iris.ExecutionOptions{Force: true},
})
```

## Convert `http.Handler/HandlerFunc`

However you are not limited to them - you are free to use any third-party middleware that is compatible with the [net/http](https://golang.org/pkg/net/http/) package.

Iris, unlike others, is 100% compatible with the standards and that's why the majority of big companies that adapt Go to their workflow, like a very famous US Television Network, trust Iris; it's up-to-date and it will be always aligned with the std `net/http` package which is modernized by the Go Authors on each new release of the Go Programming Language.

Any third-party middleware that written for `net/http` is compatible with Iris using the `iris.FromStd(aThirdPartyMiddleware)`. Remember, `ctx.ResponseWriter()` and `ctx.Request()` returns the same `net/http` input arguments of an [http.Handler](https://golang.org/pkg/net/http/#Handler).

* [From func(w http.ResponseWriter, r \*http.Request, next http.HandlerFunc)](https://github.com/kataras/iris/tree/master/_examples/convert-handlers/negroni-like/main.go)
* [From http.Handler or http.HandlerFunc](https://github.com/kataras/iris/tree/master/_examplesconvert-handlers/nethttp/main.go)
* [From func(http.HandlerFunc) http.HandlerFunc](https://github.com/kataras/iris/tree/master/_examplesconvert-handlers/real-usecase-raven/writing-middleware/main.go)

Here is a list of some handlers made specifically for Iris:

## Built-in

| Middleware | Example |
| :--- | :--- |
| [basic authentication](https://github.com/kataras/iris/tree/master/middleware/basicauth) | [iris/\_examples/auth/basicauth](https://github.com/kataras/iris/tree/master/_examples/auth/basicauth) |
| [request logger](https://github.com/kataras/iris/tree/master/middleware/logger) | [iris/\_examples/logging/request-logger](https://github.com/kataras/iris/tree/master/_examples/logging/request-logger) |
| [HTTP method override](https://github.com/kataras/iris/tree/master/middleware/methodoverride) | [iris/middleware/methodoverride/methodoverride\_test.go](https://github.com/kataras/iris/blob/master/middleware/methodoverride/methodoverride_test.go) |
| [profiling (pprof)](https://github.com/kataras/iris/tree/master/middleware/pprof) | [iris/\_examples/pprof](https://github.com/kataras/iris/tree/master/_examples/pprof) |
| [Google reCAPTCHA](https://github.com/kataras/iris/tree/master/middleware/recaptcha) | [iris/\_examples/auth/recaptcha](https://github.com/kataras/iris/tree/master/_examples/auth/recaptcha) |
| [hCaptcha](https://github.com/kataras/iris/tree/master/middleware/hcaptcha) | [iris/\_examples/auth/recaptcha](https://github.com/kataras/iris/tree/master/_examples/auth/hcaptcha) |
| [recovery](https://github.com/kataras/iris/tree/master/middleware/recover) | [iris/\_examples/recover](https://github.com/kataras/iris/tree/master/_examples/recover) |
| [rate](https://github.com/kataras/iris/tree/master/middleware/rate) | [iris/\_examples/request-ratelimit](https://github.com/kataras/iris/tree/master/_examples/request-ratelimit) |
| [jwt](https://github.com/kataras/iris/tree/master/middleware/jwt) | [iris/\_examples/auth/jwt](https://github.com/kataras/iris/tree/master/_examples/auth/jwt) |
| [requestid](https://github.com/kataras/iris/tree/master/middleware/requestid) | [iris/middleware/requestid/requestid\_test.go](https://github.com/kataras/iris/blob/master/_examples/middleware/requestid/requestid_test.go) |

## Community

| Middleware | Description | Example |
| :--- | :--- | :--- |
| [jwt](https://github.com/iris-contrib/middleware/tree/master/jwt) | Middleware checks for a JWT on the `Authorization` header on incoming requests and decodes it | [iris-contrib/middleware/jwt/\_example](https://github.com/iris-contrib/middleware/tree/master/jwt/_example) |
| [cors](https://github.com/iris-contrib/middleware/tree/master/cors) | HTTP Access Control | [iris-contrib/middleware/cors/\_example](https://github.com/iris-contrib/middleware/tree/master/cors/_example) |
| [secure](https://github.com/iris-contrib/middleware/tree/master/secure) | Middleware that implements a few quick security wins | [iris-contrib/middleware/secure/\_example](https://github.com/iris-contrib/middleware/tree/master/secure/_example/main.go) |
| [tollbooth](https://github.com/iris-contrib/middleware/tree/master/tollboothic) | Generic middleware to rate-limit HTTP requests | [iris-contrib/middleware/tollboothic/\_examples/limit-handler](https://github.com/iris-contrib/middleware/tree/master/tollboothic/_examples/limit-handler) |
| [cloudwatch](https://github.com/iris-contrib/middleware/tree/master/cloudwatch) | AWS cloudwatch metrics middleware | [iris-contrib/middleware/cloudwatch/\_example](https://github.com/iris-contrib/middleware/tree/master/cloudwatch/_example) |
| [new relic](https://github.com/iris-contrib/middleware/tree/master/newrelic) | Official [New Relic Go Agent](https://github.com/newrelic/go-agent) | [iris-contrib/middleware/newrelic/\_example](https://github.com/iris-contrib/middleware/tree/master/newrelic/_example) |
| [prometheus](https://github.com/iris-contrib/middleware/tree/master/prometheus) | Easily create metrics endpoint for the [prometheus](http://prometheus.io) instrumentation tool | [iris-contrib/middleware/prometheus/\_example](https://github.com/iris-contrib/middleware/tree/master/prometheus/_example) |
| [casbin](https://github.com/iris-contrib/middleware/tree/master/casbin) | An authorization library that supports access control models like ACL, RBAC, ABAC | [iris-contrib/middleware/casbin/\_examples](https://github.com/iris-contrib/middleware/tree/master/casbin/_examples) |
| [raven](https://github.com/iris-contrib/middleware/tree/master/raven) | Sentry client in Go | [iris-contrib/middleware/raven/\_example](https://github.com/iris-contrib/middleware/blob/master/raven/_example/main.go) |
| [csrf](https://github.com/iris-contrib/middleware/tree/master/csrf) | Cross-Site Request Forgery Protection | [iris-contrib/middleware/csrf/\_example](https://github.com/iris-contrib/middleware/blob/master/csrf/_example/main.go) |
| [go-i18n](https://github.com/iris-contrib/middleware/tree/master/go-i18n) | i18n Iris Loader for nicksnyder/go-i18n | [iris-contrib/middleware/go-i18n/\_example](https://github.com/iris-contrib/middleware/blob/master/go-i18n/_example/main.go) |
| [throttler](https://github.com/iris-contrib/middleware/tree/master/throttler) | Rate limiting access to HTTP endpoints | [iris-contrib/middleware/throttler/\_example](https://github.com/iris-contrib/middleware/blob/master/throttler/_example/main.go) |

## Third-Party

Iris has its own middleware form of `func(ctx iris.Context)` but it's also compatible with all `net/http` middleware forms. See [here](https://github.com/kataras/iris/tree/master/_examples/convert-handlers).


<!-- slide:break-100 -->
