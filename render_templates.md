# Templates

Iris gives you the freedom to render templates through  [html/template](https://golang.org/pkg/html/template) or Django-syntax package [flosch/pongo2](https://github.com/flosch/pongo2) , via **config.Iris.Render.Template = config.Template{} / iris.Config().Render.Template = config.Template{}**.

Template engine's HTML result  minified automatically for you but you can change it via `iris.Config().Render.Template.Minify = false ` 


- `HTMLTemplate` is the html/template 
-  `Pongo` is the flosch/pongo2
----

A snippet:
```go

iris.Get("/default_standar", func(ctx *iris.Context){
  ctx.Render("index.html",nil) // this will render ./templates/index.html
})

```

Let's read and learn how to set the configuration now.
```go
import (
    "github.com/kataras/iris/config"
    //...
)

```

```go
// These are the defaults
templateConfig := config.Template {
  // iris.DefaultEngine is the iris.HTMLEngine  or iris.PongoEngine
  Engine:  config.DefaultEngine
  // Common options for all template engines 
  Gzip:          false,
  Minify:        true, // minify the html before server's listen
  IsDevelopment: false,
  Directory:     "templates",
  Extensions:    []string{".html"},
  ContentType:   "text/html",
  Charset:       "UTF-8",
  Layout:        "", 
  Asset:         nil, // func(name string) ([]byte, error)
  AssetNames:    nil, // func() []string
  
  // Options when you're using html/template | When Engine == config.HTMLTemplate
  HTMLTemplate: config.HTMLTemplate {
    Left: "{{", 
    Right: "}}",
    Funcs: make([]template.FuncMap, 0),
   },
   
  // Option when you're using pongo2 | When Engine == config.PongoEngine
  Pongo:  config.Pongo{Filters: make(map[string]pongo2.FilterFunction, 0)}
}

// Set

// 1. Directly via complete custom configuration field
iris.Config().Render.Template = templateConfig

// 2. Fast way - Pongo snippet
iris.Config().Render.Template.Engine = iris.PongoEngine
iris.Config().Render.Template.Directory = "mytemplates"
iris.Config().Render.Template.Pongo.Filters = ...

// 3. Fast way - HTMLTemplate snippet
iris.Config().Render.Template.Engine = iris.HTMLTemplate // or iris.DefaultEngine
iris.Config().Render.Template.Layout = "layout/layout.html" // = ./templates/layout/layout.html
//...
 
// 4.
theDefaults := config.DefaultTemplate()
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

#### `HTMLTemplate`

```go
// main.go

package main

import (
	"github.com/kataras/iris"
)

type mypage struct {
  Message string
}

func main() {
	iris.Config().Render.Template.Layout = "layouts/layout.html"
	iris.Get("/", func(ctx *iris.Context) {
		if err := ctx.Render("page1.html", mypage{"Message from page1!"}); err != nil {
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

<h1> The message: {{.Message}} </h1>

{{ render "partials/page1_partial1.html"}}

</div>
```

```html
<!-- templates/partials/page1_partial1.html -->

<div style="background-color:white;color:red"> <h1> Page 1's Partial 1 </h1> </div>
```

Run main.go open browser and navigate to the localhost:8080 -> view page source, this is the **output**: 

```html
<!-- OUTPUT -->
<html>
  <head>
    <title>My Layout</title>
  </head>
  <body>
    
    <div style="background-color:black;color:blue">

    <h1> The message: Message from page1! </h1>

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
    "github.com/kataras/iris/config"
)

func main() {

	iris.Config().Render.Template.Engine = config.PongoEngine // or iris.PongoEngine without need to import the config

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

Run main.go open browser and navigate to the localhost:8080 -> view page source, this is the **output**: 
```html
<!-- OUTPUT -->
<html>
<head><title>Hello Pongo2 from Iris</title></head>
<body>
	 <p>iris is an admin!</p>
</body>
</html>
```