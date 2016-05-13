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

Let's read and learn how to set the configuration now.

```go
// These are the defaults
templateConfig := &iris.TemplateConfig {
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

// Set

// 1. Directly via complete custom configuration
iris.Config().Templates = templateConfig

// 2. Fast way - Pongo snippet
iris.Config().Templates.Engine = iris.PongoEngine
iris.Config().Templates.Directory = "mytemplates"
iris.Config().Templates.Pongo.Filters = ...

// 3. Fast way - Standar snippet
iris.Config().Templates.Engine = iris.StandarEngine
iris.Config().Templates.Layout = "layout/layout.html" //./templates/layout/layout.html
//...
 
// 4.
theDefaults := iris.Config().Templates
theDefaults.Extensions = []string{".myExtension"}
//...
```

```go
// HTML builds up the response from the specified template and bindings.
HTML(status int, name string, binding interface{}, layout ...string) error
// Render same as .HTML but with status to iris.StatusOK (200)
Render(name string, binding interface{}, layout ...string) error

```

### Examples

#### `Standar`

```go
// main.go

package main

import (
	"github.com/kataras/iris"
)

func main() {
	iris.Config().Templates.Layout = "layouts/layout.html"
	iris.Get("/", func(ctx *iris.Context) {
		if err := ctx.Render("page1.html", nil); err != nil {
			panic(err)
		}
	})

	println("Server is running at: 8080")
	iris.Listen(":8080")
}

```

```html
<!-- templates/layouts/layout.html -->

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
<!-- templates/page1.html -->

<div style="background-color:black;color:blue">

<h1> Page 1 </h1>

{{ render "partials/page1_partial1.html"}}

</div>
```

```html
<!-- templates/partials/page1_partial1.html -->

<div style="background-color:white;color:red"> <h1> Page 1's Partial 1 </h1> </div>
```

Run main.go open browser and navigate to the localhost:8080, this is the **ouput**

```html
<!-- OUTPUT -->
<html>
  <head>
    <title>My Layout</title>
  </head>
  <body>
    
    <div style="background-color:black;color:blue">

    <h1> Page 1 </h1>

    <div style="background-color:white;color:red">
        <h1> Page 1's Partial 1 </h1> </div>
    </div>

  </body>
</html>
```

#### `Pongo`

```go
// main.go
package main

import (
	"github.com/kataras/iris"
)

func main() {

	iris.Config().Templates.Engine = iris.PongoEngine

	iris.Get("/", func(ctx *iris.Context) {

		err := ctx.Render("index.html", map[string]interface{}{"username": "iris", "is_admin": true})
		// OR
		//err := ctx.Render("index.html", pongo2.Context{"username": "iris", "is_admin": true})

		if err != nil {
			panic(err)
		}
	})

	println("Server is running at :8080")
	iris.Listen(":8080")
}

```

```html
<!-- templates/index.html -->

<html>
<head><title>Hello Pongo2 from Iris</title></head>
<body>
	 {% if is_admin %}<p>{{username}} is an admin!</p>{% endif %}
</body>
</html>

```