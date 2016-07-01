# Templates

Iris gives you the freedom to render templates through  **html/template**, Django-syntax package **Pongo2**, Raw **Markdown**, **Amber**, **Jade** or **Handlebars**  via ** iris.Config().Render.Template.Engine = iris.___Engine**.


- `iris.HTMLEngine` is the [html/template](https://golang.org/pkg/html/template) 
-  `iris.PongoEngine` is the [flosch/pongo2](https://github.com/flosch/pongo2)
-  `iris.AmberEngine` is the [eknkc/amber](https://github.com/eknkc/amber)
-  `iris.JadeEngine` is the [Joker/jade](https://github.com/Joker/jade)
-  `iris.Handlebars` is the [aymerick/raymond](https://github.com/aymerick/raymond)
-  `iris.MarkdownEngine`

----

```go
// RenderWithStatus builds up the response from the specified template and bindings.
RenderWithStatus(status int, name string, binding interface{}, layout ...string) error

// Render same as .RenderWithStatus but with status to iris.StatusOK (200)
Render(name string, binding interface{}, layout ...string) error 

// TemplateString same as Render but instead of client render, returns the result 
TemplateString(name string, binding interface{}, layout ...string) (string,error)

// Render same as .Render but
// returns 500 internal server error and logs the error if parse failed
MustRender(name string, binding interface{}, layout ...string) 

```

A snippet:
```go

iris.Get("/default_standar", func(ctx *iris.Context){
  ctx.Render("index.html", nil) // this will render the file ./templates/index.html
})
```
Let's read and learn how to set the configuration now.
```go

import (
    "github.com/kataras/iris/config"
    //...
)
// These are the defaults
templateConfig := config.Template {
		Engine:        DefaultEngine, //or HTMLTemplate
		Gzip:          false,
		IsDevelopment: false,
		Directory:     "templates",
		Extensions:    []string{".html"},
		ContentType:   "text/html",
		Charset:       "UTF-8",
		Layout:        "", // currently this is the only config which not working for pongo2 yet but I will find a way
		HTMLTemplate:  HTMLTemplate{Left: "{{", Right: "}}", Funcs: make(map[string]interface{}, 0), LayoutFuncs: make(map[string]interface{}, 0)},
		Jade:          Jade{Left: "{{", Right: "}}", Funcs: make(map[string]interface{}, 0), LayoutFuncs: make(map[string]interface{}, 0)},
		Pongo:         Pongo{Filters: make(map[string]pongo2.FilterFunction, 0), Globals: make(map[string]interface{}, 0)},
		Markdown:      Markdown{Sanitize: false},
		Amber:         Amber{Funcs: template.FuncMap{}},
		Handlebars:    Handlebars{Helpers: make(map[string]interface{}, 0)},
}

// Set

// 1. Directly via complete custom configuration field
iris.Config.Render.Template = templateConfig

// 2. Fast way - Pongo snippet
iris.Config.Render.Template.Engine = iris.PongoEngine
iris.Config.Render.Template.Directory = "mytemplates"
iris.Config.Render.Template.Pongo.Filters = ...

// 3. Fast way - HTMLTemplate snippet
iris.Config.Render.Template.Engine = iris.HTMLTemplate // or iris.DefaultEngine
iris.Config.Render.Template.Layout = "layout/layout.html" // = ./templates/layout/layout.html
//...
 
// 4.
theDefaults := config.DefaultTemplate()
theDefaults.Extensions = []string{".myExtension"}
//...

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
	iris.Config.Render.Template.Layout = "layouts/layout.html" // default ""
	iris.Get("/", func(ctx *iris.Context) {
		 ctx.MustRender("page1.html", mypage{"Message from page1!"})
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

	iris.Config.Render.Template.Engine = config.PongoEngine // or iris.PongoEngine without need to import the config

	iris.Get("/", func(ctx *iris.Context) {
		ctx.MustRender("index.html", map[string]interface{}{"username": "iris", "is_admin": true})
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

#### `Markdown`

```go
// main.go
package main

import (
	"github.com/kataras/iris"
	"github.com/kataras/iris/config"
)

func main() {
	// Markdown engine doesn't supports Layout and context binding
	iris.Config.Render.Template.Engine = config.MarkdownEngine
	iris.Config.Render.Template.Extensions = []string{".md"}
    
	iris.Get("/", func(ctx *iris.Context) {

		err := ctx.Render("index.md", nil) // doesnt' supports any context binding, just pure markdown
		if err != nil {
			panic(err)
		}
	})

	println("Server is running at :8080")
	iris.Listen(":8080")
}

```

```
<!-- templates/index.md -->
## Hello Markdown from Iris

This is an example of Markdown with Iris



Features
--------

All features of Sundown are supported, including:

*   **Compatibility**. The Markdown v1.0.3 test suite passes with
    the `--tidy` option.  Without `--tidy`, the differences are
    mostly in whitespace and entity escaping, where blackfriday is
    more consistent and cleaner.

*   **Common extensions**, including table support, fenced code
    blocks, autolinks, strikethroughs, non-strict emphasis, etc.

*   **Safety**. Blackfriday is paranoid when parsing, making it safe
    to feed untrusted user input without fear of bad things
    happening. The test suite stress tests this and there are no
    known inputs that make it crash.  If you find one, please let me
    know and send me the input that does it.

    NOTE: "safety" in this context means *runtime safety only*. In order to
    protect yourself against JavaScript injection in untrusted content, see
    [this example](https://github.com/russross/blackfriday#sanitize-untrusted-content).

*   **Fast processing**. It is fast enough to render on-demand in
    most web applications without having to cache the output.

*   **Thread safety**. You can run multiple parsers in different
    goroutines without ill effect. There is no dependence on global
    shared state.

*   **Minimal dependencies**. Blackfriday only depends on standard
    library packages in Go. The source code is pretty
    self-contained, so it is easy to add to any project, including
    Google App Engine projects.

*   **Standards compliant**. Output successfully validates using the
    W3C validation tool for HTML 4.01 and XHTML 1.0 Transitional.

     
 ```  


Run main.go open browser and navigate to the localhost:8080 -> view page source, this is the **output**: 

```html
<!-- OUTPUT -->

<h2>Hello Markdown from Iris</h2>

<p>This is an example of Markdown with Iris</p>

<h2>Features</h2>

<p>All features of Sundown are supported, including:
* <strong>Compatibility</strong>. The Markdown v1.0.3 test suite passes with
the <code>--tidy</code> option. Without <code>--tidy</code>, the differences are
mostly in whitespace and entity escaping, where blackfriday is
more consistent and cleaner.
* <strong>Common extensions</strong>, including table support, fenced code
blocks, autolinks, strikethroughs, non-strict emphasis, etc.
* <strong>Safety</strong>. Blackfriday is paranoid when parsing, making it safe
to feed untrusted user input without fear of bad things
happening. The test suite stress tests this and there are no
known inputs that make it crash. If you find one, please let me
know and send me the input that does it.
NOTE: &ldquo;safety&rdquo; in this context means <em>runtime safety only</em>. In order to
protect yourself against JavaScript injection in untrusted content, see
<a href="https://github.com/russross/blackfriday#sanitize-untrusted-content">this example</a>.
* <strong>Fast processing</strong>. It is fast enough to render on-demand in
most web applications without having to cache the output.
* <strong>Thread safety</strong>. You can run multiple parsers in different
goroutines without ill effect. There is no dependence on global
shared state.
* <strong>Minimal dependencies</strong>. Blackfriday only depends on standard
library packages in Go. The source code is pretty
self-contained, so it is easy to add to any project, including
Google App Engine projects.
* <strong>Standards compliant</strong>. Output successfully validates using the
W3C validation tool for HTML 4.01 and XHTML 1.0 Transitional.</p>

```

#### `Amber`

```go
// main.go
package main

import "github.com/kataras/iris"

func main() {

	iris.Config.Render.Template.Engine = iris.AmberEngine
	iris.Config.Render.Template.Extensions = []string{".amber"} 
    // this is optionally, you can just leave it to default which is .html

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Render("basic.amber", map[string]string{"Name": "iris"})

	})

	iris.Listen(":8080")
}


```

```html
<!-- templates/basic.amber -->

!!! 5
html
    head
        title Hello Amber from Iris

        meta[name="description"][value="This is a sample"]

        script[type="text/javascript"]
            var hw = "Hello #{Name}!"
            alert(hw)

        style[type="text/css"]
            body {
                background: maroon;
                color: white
            }

    body
        header#mainHeader
            ul
                li.active
                    a[href="/"] Main Page
                        [title="Main Page"]
            h1
                 | Hi #{Name}

        footer
            | Hey
            br
            | There

```

Run main.go open browser and navigate to the localhost:8080 -> view page source, this is the **output**: 
```html
<!-- OUTPUT -->
<!DOCTYPE html>
<html>
	<head>
		<title>Hello Amber from Iris</title>
		<meta name="description" value="This is a sample" />
		<script type="text/javascript">
			var hw = "Hello iris!"
			alert(hw)
		</script>
		<style type="text/css">
			body {
				background: maroon;
				color: white
			}
		</style>
	</head>
	<body>
		<header id="mainHeader">
			<ul>
				<li class="active">
					<a href="/" title="Main Page">Main Page</a>
				</li>
			</ul>
			<h1>Hi iris</h1>
		</header>
		<footer>
			Hey
			<br />
			There
		</footer>
	</body>
</html>

```

#### `Jade`

```go
// main.go
package main

import (
	"github.com/kataras/iris"
)

type Person struct {
	Name   string
	Age    int
	Emails []string
	Jobs   []*Job
}

type Job struct {
	Employer string
	Role     string
}

func main() {
	iris.Config.Render.Template.Extensions = []string{".jade"} 
    // this is optionally, you can keep .html extension
	iris.Config.Render.Template.Engine = iris.JadeEngine

	iris.Get("/", func(ctx *iris.Context) {

		job1 := Job{Employer: "Super Employer", Role: "Team leader"}
		job2 := Job{Employer: "Fast Employer", Role: "Project managment"}

		person := Person{
			Name:   "name1",
			Age:    50,
			Emails: []string{"email1@something.gr", "email2.anything@gmail.com"},
			Jobs:   []*Job{&job1, &job2},
		}

		ctx.MustRender("page.jade", person)
	})

	iris.Listen(":8080")
}

```

```html
<!-- templates/page.jade -->

doctype html
html(lang=en)
	head
		meta(charset=utf-8)
		title Title
	body
		p ads
		ul
			li The name is {{.Name}}.
			li The age is {{.Age}}.

		range .Emails
			div An email is {{.}}

		with .Jobs
			range .
				div.
				 An employer is {{.Employer}}
				 and the role is {{.Role}}

```

Run main.go open browser and navigate to the localhost:8080 -> view page source, this is the **output**: 
```html
<!-- OUTPUT -->

<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <title>Title</title>
    </head>
    <body>
        <p>ads</p>
        <ul>
            <li>The name is name1.</li>
            <li>The age is 50.</li>
        </ul>
        
            <div>An email is email1@something.gr</div>
        
            <div>An email is email2.anything@gmail.com</div>
        
        
            
                <div>
                 An employer is Super Employer
                 and the role is Team leader
                </div>
            
                <div>
                 An employer is Fast Employer
                 and the role is Project managment
                </div>
            
        
    </body>
</html>

```


#### `Handlebars`
For a more complete example with party, no layout, different layouts and partials go [here](https://github.com/iris-contrib/examples/tree/master/templates_handlebars).

```go
// main.go
//Package main a basic and simple example on how to use handlebars with Iris

package main

import (
	"github.com/aymerick/raymond"

	"github.com/kataras/iris"
)

func main() {
	// set the template engine
	iris.Config.Render.Template.Engine = iris.HandlebarsEngine

	// optionaly set handlebars helpers by importing "github.com/aymerick/raymond" when you need to return and render html
	iris.Config.Render.Template.Handlebars.Helpers["boldme"] = func(input string) raymond.SafeString {
		return raymond.SafeString("<b> " + input + "</b>")
	}

	// NOTE:
	// the Iris' route framework {{url "my-routename" myparams}} and {{urlpath "my-routename" myparams}} are working like all other template engines,
	// so  avoid custom url and urlpath helpers.

	iris.Get("/", func(ctx *iris.Context) {
		// optionally, set a context  for the template
		mycontext := iris.Map{"Name": "Iris", "Type": "Web"}

		ctx.Render("home.html", mycontext)
	})
	iris.Listen(":8080")
}

/*
MORE DOCS CAN BE FOUND HERE: https://github.com/aymerick/raymond
*/

```

```html
<!-- templates/home.html -->

<html>
  <head>
    <title>My Home page</title>

  </head>
  <body>
	Name: {{boldme Name}} <br/>
	Type: {{boldme Type}}
  </body>
</html>

```

Run main.go open browser and navigate to the localhost:8080 -> view page source, this is the **output**: 
```html
<!-- OUTPUT -->

<html>
  <head>
    <title>My Home page</title>

  </head>
  <body>
	Name: <b> Iris</b> <br/>
	Type: <b> Web</b>
  </body>
</html>

```