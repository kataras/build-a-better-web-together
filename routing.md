# Routing

As mentioned in the [Handlers](handlers.md) chapter, Iris provides several handler registration methods, each of which returns a [`RouteNameFunc`](https://godoc.org/github.com/kataras/iris#RouteNameFunc) type.

## Route naming

Route naming is easy, since we just call the returned `RouteNameFunc` with a string parameter to define a name:

```go
package main

import (
	"github.com/kataras/iris"
)

func main() {

	// define a function
	render := func(ctx *iris.Context) {
		ctx.Render("index.html", nil)
	}

	// handler registration and naming
	iris.Get("/", render)("home")
	iris.Get("/about", render)("about")
	iris.Get("/page/:id", render)("page")

	iris.Listen(":8080")
}
```

## Route reversing AKA generating URLs from the route name

When we register the handlers for a specific path, we get the ability to create URLs based on the structured data we pass to Iris. In the example above, we've named three routers, one of which even takes parameters. If we're using the default `html/template` templating engine, we can use a simple action to reverse the routes (and generae actual URLs):

```
Home: {{ url "home" }}
About: {{ url "about" }}
Page 17: {{ url "page" "17" }}
```

Above code would generate the following output:

```
Home: http://0.0.0.0:8080/ 
About: http://0.0.0.0:8080/about
Page 17: http://0.0.0.0:8080/page/17
```

## Using route names in code

We can use the following methods/functions to work with named routes (and their parameters):

* global [`Lookups`](https://godoc.org/github.com/kataras/iris#Lookups) function to get all registered routes
* [`Lookup(routeName string)`](https://godoc.org/github.com/kataras/iris#Framework.Lookup) framework method to retrieve a route by name
* [`URL(routeName string, args ...interface{})`](https://godoc.org/github.com/kataras/iris#Framework.URL) framework method to generate url string based on supplied parameters
* [`Path(routeName string, args ...interface{}`](https://godoc.org/github.com/kataras/iris#Framework.Path) framework method to generate just the path (without host and protocol) portion of the URL based on provided values
* [`RedirectTo(routeName string, args ...interface{})`](https://godoc.org/github.com/kataras/iris#Context.RedirectTo) context method to return a redirect response to a URL defined by the named route and optional parameters

## Examples

Check out the [`template_engines/template_html_4`](https://github.com/iris-contrib/examples/blob/master/template_engines/template_html_4/) example in the `iris-contrib/examples` repository.