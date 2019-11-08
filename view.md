# View

Iris offers support for **6 template parsers out of the box** through its universal [View Engine](https://godoc.org/github.com/kataras/iris/view#Engine). Of course developers can still use various go template parsers as the `Context.ResponseWriter()` completes the `http.ResponseWriter` and `io.Writer`.

Iris puts some common rules and features that their original parsers don't support by default. For example we have support for `yield`, `render`, `render_r`, `current`, `urlpath` template funcs and `Layouts` and `binding` across middlewares and **embedded template files** for all Engines.

To use unique features of a template engine you have to learn the characteristics and the syntax by reading its documentation (click the links below). Choose what fits your app's needs the most.

Let's see the list of the built-in view engines:

| Engine | Declaration | Underline Template Parser |
| :--- | :--- | :--- |
| std template/html | `iris.HTML(...)` | [html/template](https://golang.org/pkg/html/template/) package |
| django | `iris.Django(...)` | [flosch/pongo2](https://github.com/flosch/pongo2) package |
| handlebars | `iris.Handlebars(...)` | [Joker/jade](https://github.com/Joker/jade) package |
| amber | `iris.Amber(...)` | [aymerick/raymond](https://github.com/aymerick/raymond) package |
| pug(jade) | `iris.Pug(...)` | [eknkc/amber](https://github.com/eknkc/amber) package |
| jet | `iris.Jet(...)` | [CloudyKit/jet](https://github.com/CloudyKit/jet) package |

One or more view engines can be registered in the same Application. To **register** a view engine use the `RegisterView(ViewEngine)` method.

Load all templates from the "./views" folder where extension is ".html" and parse them using the standard `html/template` package.

```go
// [app := iris.New...]
tmpl := iris.HTML("./views", ".html")
app.RegisterView(tmpl)
```

To **render or execute** a view use the `Context.View` method inside the main route's handler.

```go
ctx.View("hi.html")
```

To **bind** Go values with key-value pattern inside a view through middleware or main handler use the `Context.ViewData` method before the `Context.View` one.

Bind: `{{.message}}` with `"Hello world!"`.

```go
ctx.ViewData("message", "Hello world!")
```

To bind a Go **model** to a view you have two options:

**1.**

```go
ctx.ViewData("user", User{})

// variable binding as {{.user.Name}}
```

**2.**

```go
ctx.View("user-page.html", User{})

// root binding as {{.Name}}
```

To **add a template function** use the `AddFunc` method of the preferred view engine.

```go
//       func name, input arguments, render value
tmpl.AddFunc("greet", func(s string) string {
    return "Greetings " + s + "!"
})
```

To **reload on local file changes** call the view engine's `Reload` method.

```go
tmpl.Reload(true)
```

To use **embedded** templates and not depend on local file system use the [go-bindata](https://github.com/go-bindata/go-bindata) external tool and pass its `Asset` and `AssetNames` functions to the `Binary` method of the preferred view engine.

```go
tmpl.Binary(Asset, AssetNames)
```

Example Code:

Please read the _comments_ too.

```go
// file: main.go
package main

import "github.com/kataras/iris/v12"

func main() {
    app := iris.New()

    // Parse all templates from the "./views" folder
    // where extension is ".html" and parse them
    // using the standard `html/template` package.
    tmpl := iris.HTML("./views", ".html")

    // Enable re-build on local template files changes.
    tmpl.Reload(true)

    // Default template funcs are:
    //
    // - {{ urlpath "myNamedRoute" "pathParameter_ifNeeded" }}
    // - {{ render "header.html" }}
    // and partial relative path to current page:
    // - {{ render_r "header.html" }} 
    // - {{ yield }}
    // - {{ current }}
    // Register a custom template func:
    tmpl.AddFunc("greet", func(s string) string {
        return "Greetings " + s + "!"
    })

    // Register the view engine to the views,
    // this will load the templates.
    app.RegisterView(tmpl)

    // Method:    GET
    // Resource:  http://localhost:8080
    app.Get("/", func(ctx iris.Context) {
        // Bind: {{.message}} with "Hello world!"
        ctx.ViewData("message", "Hello world!")
        // Render template file: ./views/hi.html
        ctx.View("hi.html")
    })

    app.Run(iris.Addr(":8080"))
}
```

```markup
<!-- file: ./views/hi.html -->
<html>
<head>
    <title>Hi Page</title>
</head>
<body>
    <h1>{{.message}}</h1>
    <strong>{{greet "to you"}}</strong>
</body>
</html>
```

Open a browser tab at [http://localhost:8080](http://localhost:8080).

The **rendered result** will look like this:

```markup
<html>
<head>
    <title>Hi Page</title>
</head>
<body>
    <h1>Hello world!</h1>
    <strong>Greetings to you!</strong>
</body>
</html>
```

