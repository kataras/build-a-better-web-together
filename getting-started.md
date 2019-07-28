# Getting Started

Iris has an expressive syntax for routing which feels like home. The routing algorithm is powered by the [muxie project](https://github.com/kataras/muxie) which handles requests and matches routes faster than its alternatives like httprouter and gin or echo.

Let's get started without any hassle.

Create an empty file, let's assume its name is `example.go`, then open it and copy-paste the below code.

```go
package main

import "github.com/kataras/iris"

func main() {
    app := iris.Default()
    app.Use(myMiddleware)

    app.Handle("GET", "/ping", func(ctx iris.Context) {
        ctx.JSON(iris.Map{"message": "pong"})
    })

    // Listens and serves incoming http requests
    // on http://localhost:8080.
    app.Run(iris.Addr(":8080"))
}

func myMiddleware(ctx iris.Context) {
    ctx.Application().Logger().Infof("Runs before %s", ctx.Path())
    ctx.Next()
}
```

Start a terminal session and execute the following.

```text
# run example.go and visit http://localhost:8080/ping on browser
$ go run example.go
```

## Show me more!

Let's take a small overview of how easy is to get up and running.

```go
package main

import "github.com/kataras/iris"

func main() {
    app := iris.New()
    // Load all templates from the "./views" folder
    // where extension is ".html" and parse them
    // using the standard `html/template` package.
    app.RegisterView(iris.HTML("./views", ".html"))

    // Method:    GET
    // Resource:  http://localhost:8080
    app.Get("/", func(ctx iris.Context) {
        // Bind: {{.message}} with "Hello world!"
        ctx.ViewData("message", "Hello world!")
        // Render template file: ./views/hello.html
        ctx.View("hello.html")
    })

    // Method:    GET
    // Resource:  http://localhost:8080/user/42
    //
    // Need to use a custom regexp instead?
    // Easy;
    // Just mark the parameter's type to 'string'
    // which accepts anything and make use of
    // its `regexp` macro function, i.e:
    // app.Get("/user/{id:string regexp(^[0-9]+$)}")
    app.Get("/user/{id:uint64}", func(ctx iris.Context) {
        userID, _ := ctx.Params().GetUint64("id")
        ctx.Writef("User ID: %d", userID)
    })

    // Start the server using a network address.
    app.Run(iris.Addr(":8080"))
}
```

```html
<!-- file: ./views/hello.html -->
<html>
<head>
    <title>Hello Page</title>
</head>
<body>
    <h1>{{ .message }}</h1>
</body>
</html>
```

![overview screen](https://github.com/kataras/build-a-better-web-together/raw/master/overview_screen_1.png)

> Wanna re-start your app automatically when source code changes happens? Install the [rizla](https://github.com/kataras/rizla) tool and execute `rizla main.go` instead of `go run main.go`.

At the next section we will learn more about [Routing](routing/).

