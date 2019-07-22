# Reverse Lookups

As mentioned in the [Routing](./) chapter, Iris provides several handler registration methods, each of which returns a [`Route`](https://godoc.org/github.com/kataras/iris/core/router#Route) instance.

## Route naming

Route naming is easy, since we just call the returned `*Route` with a `Name` field to define a name:

```go
package main

import "github.com/kataras/iris"

func main() {
    app := iris.New()
    // define a function
    h := func(ctx iris.Context) {
        ctx.HTML("<b>Hi</b1>")
    }

    // handler registration and naming
    home := app.Get("/", h)
    home.Name = "home"
    // or
    app.Get("/about", h).Name = "about"
    app.Get("/page/{id}", h).Name = "page"

    app.Run(iris.Addr(":8080"))
}
```

## Route reversing AKA generating URLs from the route name

When we register the handlers for a specific path, we get the ability to create URLs based on the structured data we pass to Iris. In the example above, we've named three routers, one of which even takes parameters. If we're using the default `html/template` view engine, we can use a simple action to reverse the routes (and generae actual URLs):

```bash
Home: {{ urlpath "home" }}
About: {{ urlpath "about" }}
Page 17: {{ urlpath "page" "17" }}
```

Above code would generate the following output:

```bash
Home: http://localhost:8080/ 
About: http://localhost:8080/about
Page 17: http://localhost:8080/page/17
```

## Using route names in code

We can use the following methods/functions to work with named routes (and their parameters):

* [`GetRoutes`](https://godoc.org/github.com/kataras/iris/core/router#APIBuilder.GetRoutes) function to get all registered routes
* [`GetRoute(routeName string)`](https://godoc.org/github.com/kataras/iris/core/router#APIBuilder.GetRoute) method to retrieve a route by name
* [`URL(routeName string, paramValues ...interface{})`](https://godoc.org/github.com/kataras/iris/core/router#RoutePathReverser.URL) method to generate url string based on supplied parameters
* [`Path(routeName string, paramValues ...interface{}`](https://godoc.org/github.com/kataras/iris/core/router#RoutePathReverser.Path) method to generate just the path (without host and protocol) portion of the URL based on provided values

