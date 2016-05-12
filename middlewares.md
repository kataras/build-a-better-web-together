# Middlewares

**Quick view**

```go
// First point on the static files
iris.Static("/assets", "./public/assets", 1)

// Then declare which midleware to use (custom or not)
iris.Use(myMiddleware)
iris.UseFunc(myFunc)

// Now declare routes
iris.Get("/myroute", func(c *iris.Context) {
    // do stuff
})
iris.Get("/secondroute", myMiddlewareFunc, myRouteHandlerfunc)

// Now run our server
iris.Listen(":8080")

```


Middlewares in Iris are not complicated, imagine them as simple Handlers.
They should implement the Handler interface as well:

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
	    c.WriteText(403,"Forbidden !!")
	}

}

iris.Use(&myMiddleware{})

iris.Get("/home", func (c *iris.Context){
	c.WriteHTML(iris.StatusOK,"<h1>Hello from /home </h1>")
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
    c.Write("The last HandlerFunc is the main handler, all before that are the middlewares for this route /dashboard")
})

iris.Listen(":8080")

```

> Note that middlewares must come before route declaration.


Make use one of build'n Iris [middlewares](https://github.com/kataras/iris/tree/master/middleware), view practical [examples here](https://github.com/iris-contrib/examples)

```go
package main

import (
 "github.com/kataras/iris"
 "github.com/kataras/iris/middleware/logger"
)

type Page struct {
	Title string
}

iris.Config().Render.Directory = "./yourpath/templates"

iris.Use(logger.Logger())

iris.Get("/", func(c *iris.Context) {
		c.Render("index", Page{"My Index Title"})
})

iris.Listen(":8080")
```