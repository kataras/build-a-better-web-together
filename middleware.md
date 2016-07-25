# Middleware

**Quick view**

```go
// First point to the static files
iris.Static("/assets", "./public/assets", 1)

// Then declare which middleware to use (custom or not)
iris.Use(myMiddleware{})
iris.UseFunc(func(ctx *iris.Context){})

// Now declare routes
iris.Get("/myroute", func(c *iris.Context) {
    // do stuff
})
iris.Get("/secondroute", myMiddlewareFunc, myRouteHandlerfunc)

// Now run our server
iris.Listen(":8080")


// myMiddleware will be like that

type myMiddleware struct {
  // your 'stateless' fields here
}

func (m *myMiddleware) Serve(ctx *iris.Context){
  // ...
}

```

Middleware in Iris is not complicated, they are similar to simple Handlers.
They implement the Handler interface as well:

```go
type Handler interface {
    Serve(*Context)
}
type Middleware []Handler
```

Handler middleware example:

```go

type myMiddleware struct {}

func (m *myMiddleware) Serve(c *iris.Context){
    shouldContinueToTheNextHandler := true

    if shouldContinueToTheNextHandler {
        c.Next()
    }else{
        c.Text(403,"Forbidden !!")
    }

}

iris.Use(&myMiddleware{})

iris.Get("/home", func (c *iris.Context){
    c.HTML(iris.StatusOK,"<h1>Hello from /home </h1>")
})

iris.Listen(":8080")
```

HandlerFunc middleware example:

```go

func myMiddleware(c *iris.Context){
    c.Next()
}

iris.UseFunc(myMiddleware)

```

HandlerFunc middleware for a specific route:

```go

func mySecondMiddleware(c *iris.Context){
    c.Next()
}

iris.Get("/dashboard", func(c *iris.Context) {
    loggedIn := true
    if loggedIn {
        c.Next()
    }
}, mySecondMiddleware, func (c *iris.Context){
    c.Write("The last HandlerFunc is the main handler, everything before that is middleware for this route /dashboard")
})

iris.Listen(":8080")

```

> Note that middleware must come before route declaration.

Make use of the [middleware](https://github.com/iris-contrib/middleware), view practical [examples here](https://github.com/iris-contrib/examples)

```go
package main

import (
 "github.com/kataras/iris"
 "github.com/iris-contrib/middleware/logger"
)

type Page struct {
    Title string
}

iris.Use(logger.New(iris.Logger))

iris.Get("/", func(c *iris.Context) {
    c.Render("index.html", Page{"My Index Title"})
})

iris.Listen(":8080")
```


**Done/DoneFunc**

```go
package main

import "github.com/kataras/iris"

func firstMiddleware(ctx *iris.Context) {
	ctx.Write("1. This is the first middleware, before any of route's handlers \n")
	ctx.Next()
}

func secondMiddleware(ctx *iris.Context) {
	ctx.Write("2. This is the second middleware, before the / main handler \n")
	ctx.Next()
}

func thirdMiddleware(ctx *iris.Context) {
	ctx.Write("3. This is the 3rd middleware, after the main handler \n")
	ctx.Next()
}

func lastAlwaysMiddleware(ctx *iris.Context) {
	ctx.Write("4. This is the ALWAYS LAST Handler \n")
}

func main() {

	iris.UseFunc(firstMiddleware)
	iris.DoneFunc(lastAlwaysMiddleware)

	iris.Get("/", secondMiddleware, func(ctx *iris.Context) {
		ctx.Write("Hello from / \n")
		ctx.Next() // .Next because we 're using the third middleware after that, and lastAlwaysMiddleware also
	}, thirdMiddleware)
	iris.Listen(":8080")

}


```

**Done/DoneFunc with Parties**

```go
// Package main same as middleware_2 but with party
package main

import "github.com/kataras/iris"

func firstMiddleware(ctx *iris.Context) {
	ctx.Write("1. This is the first middleware, before any of route's handlers \n")
	ctx.Next()
}

func secondMiddleware(ctx *iris.Context) {
	ctx.Write("2. This is the second middleware, before the / main handler \n")
	ctx.Next()
}

func thirdMiddleware(ctx *iris.Context) {
	ctx.Write("3. This is the 3rd middleware, after the main handler \n")
	ctx.Next()
}

func lastAlwaysMiddleware(ctx *iris.Context) {
	ctx.Write("4. This is the ALWAYS LAST Handler \n")
}

func main() {

	// with parties:
	myParty := iris.Party("/myparty", firstMiddleware).DoneFunc(lastAlwaysMiddleware)
	{
		myParty.Get("/", secondMiddleware, func(ctx *iris.Context) {
			ctx.Write("Hello from /myparty/ \n")
			ctx.Next() // .Next because we 're using the third middleware after that, and lastAlwaysMiddleware also
		}, thirdMiddleware)

	}

	iris.Listen(":8080")

}


```

> Done/DoneFuncs are just last-executed handlers, like Use/UseFunc the children party inheritates these 'done/last' handlers also.