# Templates

Iris gives you the freedom to render templates through  [html/template](https://golang.org/pkg/html/template) or Django-syntax package [flosch/pongo2](https://github.com/flosch/pongo2) , via **IrisConfig**.


- html/template is the `Standar`
- flosch/pongo2 is the `Pongo`

A snippet:
```go

iris.Get("/default_standar", func(ctx *iris.Context){
  ctx.Render("index.html",nil) // this will render ./templates/index.html
})

```

Let's read about configuration now.

```go
// These are the defaults
&iris.TemplateConfig {
  // iris.StandarEngine or iris.PongoEngine
  Engine:  iris.StandarEngine

  // Common options for all template engines 
  Gzip:          false,
  IsDevelopment: false,
  Directory:     "templates",
  Extensions:    []string{".html"},
  ContentType:   "text/html",
  Charset:       "UTF-8",
  Layout:        "", 
  Asset:         nil, // func(name string) ([]byte, error)
  AssetNames:    nil, // func() []string
  // Options when you're using pongo2 | When Engine == iris.StandarEngine
  Standar: &StandarConfig {
    Left: "{{", 
    Right: "}}",
    Funcs: make([]template.FuncMap, 0),
   },
  // Option when you're using pongo2 | When Engine == iris.PongoEngine
  Pongo:  &PongoConfig{Filters: make(map[string]pongo2.FilterFunction, 0)}
}
```

```go
// HTML builds up the response from the specified template and bindings.
HTML(status int, name string, binding interface{}, layout ...string) error
// Render same as .HTML but with status to iris.StatusOK (200)
Render(name string, binding interface{}, layout ...string) error

```

### Example

```go
//
// FILE: ./main.go
//
package main

import (
	"github.com/kataras/iris"
)

type mypage struct {
	Title   string
	Message string
}

func main() {

	//optionally - before the load.
	iris.Config().Render.Delims.Left = "${" // Default "{{"
	iris.Config().Render.Delims.Right = "}" // this will change the behavior of {{.Property}} to ${.Property}. Default "}}"
	//iris.Config().Render.Funcs = template.FuncMap(...)

	iris.Config().Render.Directory = "templates" // Default "templates"
	iris.Config().Render.Layout = "layout" // Default is ""
	iris.Config().Render.Gzip = true       // Default is false
    //...

	//or make a new renderOptions := &render.Config{...} and do iris.Config().Render = renderOptions

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Render("mypage", mypage{"My Page title", "Hello world!"}) //,"mylayout_for_this" <- optionally
	})

	println("Server is running at :8080")
	iris.Listen(":8080")
}
```

```html
<!--
 FILE: ./templates/layout.html
-->
<html>
  <head>
    <title>My Layout</title>

  </head>
  <body>
    <!-- Render the current template here -->
    {{ yield }}
  </body>
</html>

```

```html
<!--
 FILE: ./templates/mypage.html
-->
<h1> Title: {{.Title}} <h1>
<h3> Message : {{.Message}} </h3>
```