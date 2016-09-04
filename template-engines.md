## Install

Install the go-template package.

```sh
$ go get -u github.com/kataras/go-template
```

## Iris' Station configuration 

Remember, when 'station' we mean the default `iris.$CALL ` or `api:= iris.New(); api.$CALL`

```go
iris.Config.IsDevelopment = true // reloads the templates on each request, defaults to false
iris.Config.Gzip  = true // compressed gzip contents to the client, the same for Response Engines also, defaults to false
iris.Config.Charset = "UTF-8" // defaults to "UTF-8", the same for Response Engines also
```

The last two options (Gzip, Charset) can be overriden for specific 'Render' action:

```go
func(ctx *iris.Context){
    ctx.Render("templateFile.html", anyBindingStruct{}, iris.RenderOptions{"gzip":false, "charset": "UTF-8"})
}
```

## How to use

Most examples are written for the HTML Template Engine(default and built'n template engine for iris) but works for the rest of the engines also.


You will see first the template file's code, after the main.go code


**HTML Template Engine, defaulted**


```html
<!-- ./templates/hi.html -->

<html>
<head>
<title>Hi Iris [THE TITLE]</title>
</head>
<body>
	<h1>Hi {{.Name}}
</body>
</html>
```

```go
// ./main.go
package main

import "github.com/kataras/iris"

// nothing to do, defaults to ./templates and .html extension, no need to import any template engine because HTML engine is the default
// if anything else has been registered
func main() {
	iris.Config.IsDevelopment = true // this will reload the templates on each request, defaults to false
	iris.Get("/hi", hi)
	iris.Listen(":8080")
}

func hi(ctx *iris.Context) {
	ctx.MustRender("hi.html", struct{ Name string }{Name: "iris"})
}


```

```html
<!-- ./templates/layout.html -->
<html>
<head>
<title>My Layout</title>

</head>
<body>
	<h1>Body is:</h1>
	<!-- Render the current template here -->
	{{ yield }}
</body>
</html>
```

```html
 <!-- ./templates/mypage.html --> 
<h1>
	Title: {{.Title}}
</h1>
<h3>Message : {{.Message}} </h3>
```

```go
// ./main.go
package main

import (
	"github.com/kataras/go-template/html"
	"github.com/kataras/iris"
)

type mypage struct {
	Title   string
	Message string
}

func main() {

	iris.UseTemplate(html.New(html.Config{
		Layout: "layout.html",
	})).Directory("./templates", ".html") // the .Directory() is optional also, defaults to ./templates, .html
	// Note for html: this is the default iris' templaet engine, if zero engines added, then the template/html will be used automatically
	// These lines are here to show you how you can change its default configuration

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Render("mypage.html", mypage{"My Page title", "Hello world!"}, iris.RenderOptions{"gzip": true})
		// Note that: you can pass "layout" : "otherLayout.html" to bypass the config's Layout property or iris.NoLayout to disable layout on this render action.
		// RenderOptions is an optional parameter
	})

	iris.Listen(":8080")
}


```

```html
<!-- ./templates/layouts/layout.html -->
<html>
<head>
<title>Layout</title>

</head>
<body>
	<h1>This is the global layout</h1>
	<br />
	<!-- Render the current template here -->
	{{ yield }}
</body>
</html>


```

```html
<!-- ./templates/layouts/mylayout.html -->
<html>
<head>
<title>my Layout</title>

</head>
<body>
	<h1>This is the layout for the /my/ and /my/other routes only</h1>
	<br />
	<!-- Render the current template here -->
	{{ yield }}
</body>
</html>


```

```html
<!-- ./templates/partials/page1_partial1.html -->
<div style="background-color: white; color: red">
	<h1>Page 1's Partial 1</h1>
</div>


```

```html
<!-- ./templates/page1.html -->
<div style="background-color: black; color: blue">

	<h1>Page 1</h1>

	{{ render "partials/page1_partial1.html"}}

</div>


```

```go
// ./main.go
package main

import (
	"github.com/kataras/go-template/html"
	"github.com/kataras/iris"
)

func main() {
	// directory and extensions defaults to ./templates, .html for all template engines
	iris.UseTemplate(html.New(html.Config{Layout: "layouts/layout.html"}))
	//iris.Config.Render.Template.Gzip = true
	iris.Get("/", func(ctx *iris.Context) {
		if err := ctx.Render("page1.html", nil); err != nil {
			println(err.Error())
		}
	})

	// remove the layout for a specific route
	iris.Get("/nolayout", func(ctx *iris.Context) {
		if err := ctx.Render("page1.html", nil, iris.RenderOptions{"layout": iris.NoLayout}); err != nil {
			println(err.Error())
		}
	})

	// set a layout for a party, .Layout should be BEFORE any Get or other Handle party's method
	my := iris.Party("/my").Layout("layouts/mylayout.html")
	{
		my.Get("/", func(ctx *iris.Context) {
			ctx.MustRender("page1.html", nil)
		})
		my.Get("/other", func(ctx *iris.Context) {
			ctx.MustRender("page1.html", nil)
		})
	}

	iris.Listen(":8080")
}


```

```html
<!-- ./templates/layouts/layout.html -->

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
<!-- ./templates/partials/page1_partial1.html -->
<div style="background-color: white; color: red">
	<h1>Page 1's Partial 1</h1>
</div>


```

```html
<!-- ./templates/page1.html -->
<div style="background-color: black; color: blue">

	<h1>Page 1</h1>

	{{ render "partials/page1_partial1.html"}}

</div>


```

```go
// ./main.go
package main

import (
	"github.com/kataras/go-template/html"
	"github.com/kataras/iris"
)

func main() {
	// directory and extensions defaults to ./templates, .html for all template engines
	iris.UseTemplate(html.New(html.Config{Layout: "layouts/layout.html"}))

	iris.Get("/", func(ctx *iris.Context) {
		s := iris.TemplateString("page1.html", nil)
		ctx.Write("The plain content of the template is: %s", s)
	})

	iris.Listen(":8080")
}


```

```html
<!-- ./templates/page.html -->
<a href="{{url "my-page1"}}">http://127.0.0.1:8080/mypath</a>
<br />
<br />
<a href="{{url "my-page2" "theParam1" "theParam2"}}">http://127.0.0.1:8080/mypath2/:param1/:param2</a>
<br />
<br />
<a href="{{url "my-page3" "theParam1" "theParam2AfterStatic"}}">http://127.0.0.1:8080/mypath3/:param1/statichere/:param2</a>
<br />
<br />
<a href="{{url "my-page4" "theParam1" "theparam2AfterStatic" "otherParam" "matchAnything"}}">http://127.0.0.1:8080/mypath4/:param1/statichere/:param2/:otherparam/*something</a>
<br />
<br />
<a href="{{url "my-page5" "theParam1" "theParam2AfterStatic" "otherParam" "matchAnythingAfterStatic"}}">http://127.0.0.1:8080/mypath5/:param1/statichere/:param2/:otherparam/anything/*anything</a>
<br />
<br />
<a href="{{url "my-page6" .ParamsAsArray }}">http://127.0.0.1:8080/mypath6/:param1/:param2/staticParam/:param3AfterStatic</a>


```

```go
// ./main.go
// Package main an example on how to naming your routes & use the custom 'url' HTML Template Engine, same for other template engines
// we don't need to import the kataras/go-template/html because iris uses this as the default engine if no other template engine has been registered.
package main

import (
	"github.com/kataras/iris"
)

func main() {

	iris.Get("/mypath", emptyHandler)("my-page1")
	iris.Get("/mypath2/:param1/:param2", emptyHandler)("my-page2")
	iris.Get("/mypath3/:param1/statichere/:param2", emptyHandler)("my-page3")
	iris.Get("/mypath4/:param1/statichere/:param2/:otherparam/*something", emptyHandler)("my-page4")

	// same with Handle/Func
	iris.HandleFunc("GET", "/mypath5/:param1/statichere/:param2/:otherparam/anything/*anything", emptyHandler)("my-page5")

	iris.Get("/mypath6/:param1/:param2/staticParam/:param3AfterStatic", emptyHandler)("my-page6")

	iris.Get("/", func(ctx *iris.Context) {
		// for /mypath6...
		paramsAsArray := []string{"theParam1", "theParam2", "theParam3"}

		if err := ctx.Render("page.html", iris.Map{"ParamsAsArray": paramsAsArray}); err != nil {
			panic(err)
		}
	})

	iris.Get("/redirect/:namedRoute", func(ctx *iris.Context) {
		routeName := ctx.Param("namedRoute")

		println("The full uri of " + routeName + "is: " + iris.URL(routeName))
		// if routeName == "my-page1"
		// prints: The full uri of my-page1 is: http://127.0.0.1:8080/mypath
		ctx.RedirectTo(routeName)
		// http://127.0.0.1:8080/redirect/my-page1 will redirect to -> http://127.0.0.1:8080/mypath
	})

	iris.Listen(":8080")
}

func emptyHandler(ctx *iris.Context) {
	ctx.Write("Hello from %s.", ctx.PathString())

}


```


```html
<!-- ./templates/page.html -->
<!-- the only difference between normal named routes and dynamic subdomains named routes is that the first argument of  url
is the subdomain part instead of named parameter-->

<a href="{{url "dynamic-subdomain1" "username1"}}">username1.127.0.0.1:8080/mypath</a>
<br />
<br />
<a href="{{url "dynamic-subdomain2" "username2" "theParam1" "theParam2"}}">username2.127.0.0.1:8080/mypath2/:param1/:param2</a>
<br />
<br />
<a href="{{url "dynamic-subdomain3" "username3" "theParam1" "theParam2AfterStatic"}}">username3.127.0.0.1:8080/mypath3/:param1/statichere/:param2</a>
<br />
<br />
<a href="{{url "dynamic-subdomain4" "username4" "theParam1" "theparam2AfterStatic" "otherParam" "matchAnything"}}">username4.127.0.0.1:8080/mypath4/:param1/statichere/:param2/:otherparam/*something</a>
<br />
<br />
<a href="{{url "dynamic-subdomain5" .ParamsAsArray }}" >username5.127.0.0.1:8080/mypath6/:param1/:param2/staticParam/:param3AfterStatic</a>


```
I will add hosts files contens only once, here, you can imagine the rest.

**File location is Windows: Drive:/Windows/system32/drivers/etc/hosts, on Linux: /etc/hosts **
```sh
# localhost name resolution is handled within DNS itself.
127.0.0.1       localhost
::1             localhost
#-IRIS-For development machine, you have to configure your dns also for online, search google how to do it if you don't know

127.0.0.1		username1.127.0.0.1
127.0.0.1		username2.127.0.0.1
127.0.0.1		username3.127.0.0.1
127.0.0.1		username4.127.0.0.1
127.0.0.1		username5.127.0.0.1
# note that you can always use custom subdomains
#-END IRIS-

```
```go
// ./main.go
// Package main same example as template_html_4 but with wildcard subdomains
package main

import (
	"github.com/kataras/iris"
)

func main() {

	wildcard := iris.Party("*.")
	{
		wildcard.Get("/mypath", emptyHandler)("dynamic-subdomain1")
		wildcard.Get("/mypath2/:param1/:param2", emptyHandler)("dynamic-subdomain2")
		wildcard.Get("/mypath3/:param1/statichere/:param2", emptyHandler)("dynamic-subdomain3")
		wildcard.Get("/mypath4/:param1/statichere/:param2/:otherparam/*something", emptyHandler)("dynamic-subdomain4")
		wildcard.Get("/mypath5/:param1/:param2/staticParam/:param3AfterStatic", emptyHandler)("dynamic-subdomain5")
	}

	iris.Get("/", func(ctx *iris.Context) {
		// for dynamic_subdomain:8080/mypath5...
		// the first parameter is always the subdomain part
		paramsAsArray := []string{"username5", "theParam1", "theParam2", "theParam3"}

		if err := ctx.Render("page.html", iris.Map{"ParamsAsArray": paramsAsArray}); err != nil {
			panic(err)
		}
	})

	iris.Get("/redirect/:namedRoute/:subdomain", func(ctx *iris.Context) {
		routeName := ctx.Param("namedRoute")
		subdomain := ctx.Param("subdomain")
		println("The full uri of " + routeName + "is: " + iris.URL(routeName, subdomain))
		// if routeName == "dynamic-subdomain1" && subdomain == "username1"
		// prints: The full uri ofd ynamic-subdomain1 is: http://username1.127.0.0.1:8080/mypath
		ctx.RedirectTo(routeName, subdomain) // the second parameter is the arguments, the first argument for dynamic subdomains is the subdomain part, after this, the named parameters
		// http://127.0.0.1:8080/redirect/my-subdomain1 will redirect to ->  http://username1.127.0.0.1:8080/mypath
	})

	iris.Listen("127.0.0.1:8080")
}

func emptyHandler(ctx *iris.Context) {
	ctx.Write("[SUBDOMAIN: %s]Hello from Path: %s.", ctx.Subdomain(), ctx.PathString())
}


```


**Django Template Engine**

```html
<!-- ./templates/mypage.html -->
<html>
<head>
<title>Hello Django from Iris</title>

</head>
<body>
	{% if is_admin %}
	<p>{{username}} is an admin!</p>
	{% endif %}
</body>
</html>

```

```go
// ./main.go
package main

import (
	"github.com/kataras/go-template/django"
	"github.com/kataras/iris"
)

func main() {

	iris.UseTemplate(django.New()).Directory("./templates", ".html")

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Render("mypage.html", map[string]interface{}{"username": "iris", "is_admin": true}, iris.RenderOptions{"gzip": true})
	})

	iris.Listen(":8080")
}


```

```html
<!-- ./templates/page.html -->
<!-- the only difference between normal named routes and dynamic subdomains named routes is that the first argument of  url
is the subdomain part instead of named parameter-->
<a href="{{ url("dynamic-subdomain1","username1") }}">username1.127.0.0.1:8080/mypath</a>
<br />
<br />
<a href="{{ url("dynamic-subdomain2","username2","theParam1","theParam2") }}">username2.127.0.0.1:8080/mypath2/:param1/:param2</a>
<br />
<br />
<a href="{{ url("dynamic-subdomain3","username3","theParam1","theParam2AfterStatic") }}" >username3.127.0.0.1:8080/mypath3/:param1/statichere/:param2</a>
<br />
<br />
<a href="{{ url("dynamic-subdomain4","username4","theParam1","theparam2AfterStatic","otherParam","matchAnything") }}" >username4.127.0.0.1:8080/mypath4/:param1/statichere/:param2/:otherparam/*something</a>
<br />
<br />



```

```go
// ./main.go
// Package main same example as template_html_5 but for django/pongo2
package main

import (
	"github.com/kataras/go-template/django"
	"github.com/kataras/iris"
)

func main() {
	iris.UseTemplate(django.New())

	wildcard := iris.Party("*.")
	{
		wildcard.Get("/mypath", emptyHandler)("dynamic-subdomain1")
		wildcard.Get("/mypath2/:param1/:param2", emptyHandler)("dynamic-subdomain2")
		wildcard.Get("/mypath3/:param1/statichere/:param2", emptyHandler)("dynamic-subdomain3")
		wildcard.Get("/mypath4/:param1/statichere/:param2/:otherparam/*something", emptyHandler)("dynamic-subdomain4")
	}

	iris.Get("/", func(ctx *iris.Context) {
		// for dynamic_subdomain:8080/mypath5...
		// the first parameter is always the subdomain part

		if err := ctx.Render("page.html", nil); err != nil {
			panic(err)
		}
	})

	iris.Get("/redirect/:namedRoute/:subdomain", func(ctx *iris.Context) {
		routeName := ctx.Param("namedRoute")
		subdomain := ctx.Param("subdomain")
		println("The full uri of " + routeName + "is: " + iris.URL(routeName, subdomain))
		// if routeName == "dynamic-subdomain1" && subdomain == "username1"
		// prints: The full uri ofd ynamic-subdomain1 is: http://username1.127.0.0.1:8080/mypath
		ctx.RedirectTo(routeName, subdomain) // the second parameter is the arguments, the first argument for dynamic subdomains is the subdomain part, after this, the named parameters
		// http://127.0.0.1:8080/redirect/my-subdomain1 will redirect to ->  http://username1.127.0.0.1:8080/mypath
	})

	iris.Listen("127.0.0.1:8080")
}

func emptyHandler(ctx *iris.Context) {
	ctx.Write("[SUBDOMAIN: %s]Hello from Path: %s.", ctx.Subdomain(), ctx.PathString())
}


```

> Note that, you can see more django examples syntax by navigating [here](https://github.com/flosch/pongo2)


** Handlebars Template Engine **

```html
<!-- ./templates/layouts/layout.html -->

<html>
<head>
<title>Layout</title>

</head>
<body>
	<h1>This is the global layout</h1>
	<br />
	<!-- Render the current template here -->
	{{ yield }}
</body>
</html>

```

```html
<!-- ./templates/layouts/mylayout.html -->
<html>
<head>
<title>my Layout</title>

</head>
<body>
	<h1>This is the layout for the /my/ and /my/other routes only</h1>
	<br />
	<!-- Render the current template here -->
	{{ yield }}
</body>
</html>


```


```html
<!-- ./templates/partials/home_partial.html -->
<div style="background-color: white; color: red">
	<h1>Home's' Partial here!!</h1>
</div>


```

```html
<!-- ./templates/home.html -->
<div style="background-color: black; color: white">

	Name: {{boldme Name}} <br /> Type: {{boldme Type}} <br /> Path:
	{{boldme Path}} <br />
	<hr />

	The partial is: {{ render "partials/home_partial.html"}}

</div>

```


```go
// ./main.go
package main

import (
	"github.com/aymerick/raymond"
	"github.com/kataras/go-template/handlebars"
	"github.com/kataras/iris"
)

type mypage struct {
	Title   string
	Message string
}

func main() {
	// set the configuration for this template engine  (all template engines has its configuration)
	config := handlebars.DefaultConfig()
	config.Layout = "layouts/layout.html"
	config.Helpers["boldme"] = func(input string) raymond.SafeString {
		return raymond.SafeString("<b> " + input + "</b>")
	}

	// set the template engine
	iris.UseTemplate(handlebars.New(config)).Directory("./templates", ".html") // or .hbs , whatever you want

	iris.Get("/", func(ctx *iris.Context) {
		// optionally, set a context  for the template
		ctx.Render("home.html", map[string]interface{}{"Name": "Iris", "Type": "Web", "Path": "/"})

	})

	// remove the layout for a specific route using iris.NoLayout
	iris.Get("/nolayout", func(ctx *iris.Context) {
		if err := ctx.Render("home.html", nil, iris.RenderOptions{"layout": iris.NoLayout}); err != nil {
			ctx.Write(err.Error())
		}
	})

	// set a layout for a party, .Layout should be BEFORE any Get or other Handle party's method
	my := iris.Party("/my").Layout("layouts/mylayout.html")
	{
		my.Get("/", func(ctx *iris.Context) {
			// .MustRender -> same as .Render but logs the error if any and return status 500 on client
			ctx.MustRender("home.html", map[string]interface{}{"Name": "Iris", "Type": "Web", "Path": "/my/"})
		})
		my.Get("/other", func(ctx *iris.Context) {
			ctx.MustRender("home.html", map[string]interface{}{"Name": "Iris", "Type": "Web", "Path": "/my/other"})
		})
	}

	iris.Listen(":8080")
}

// Note than you can see more handlebars examples syntax by navigating to https://github.com/aymerick/raymond


```

>  Note than you can see more handlebars examples syntax by navigating [here](https://github.com/aymerick/raymond)


**Pug/Jade Template Engine**

```html
<!-- ./templates/partials/page1_partial1.jade -->
#footer
  p Copyright (c) foobar


```

```html
<!-- ./templates/page.jade -->
doctype html
html(lang=en)
	head
		meta(charset=utf-8)
		title Title
	body
		p ads
		ul
			li The name is {{bold .Name}}.
			li The age is {{.Age}}.

		range .Emails
			div An email is {{.}}

		with .Jobs
			range .
				div.
				 An employer is {{.Employer}}
				 and the role is {{.Role}}

		{{ render "partials/page1_partial1.jade"}}


```

```go
// ./main.go
package main

import (
	"html/template"

	"github.com/kataras/go-template/pug"
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
	// set the configuration for this template engine  (all template engines has its configuration)
	cfg := pug.DefaultConfig()
	cfg.Funcs["bold"] = func(content string) (template.HTML, error) {
		return template.HTML("<b>" + content + "</b>"), nil
	}

	iris.UseTemplate(pug.New(cfg)).
		Directory("./templates", ".jade")

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
<!-- ./templates/page.jade -->
a(href='{{url "dynamic-subdomain1" "username1"}}') username1.127.0.0.1:8080/mypath
p.
 a(href='{{url "dynamic-subdomain2" "username2" "theParam1" "theParam2"}}') username2.127.0.0.1:8080/mypath2/:param1/:param2

p.
 a(href='{{url "dynamic-subdomain3" "username3" "theParam1" "theParam2AfterStatic"}}') username3.127.0.0.1:8080/mypath3/:param1/statichere/:param2

p.
 a(href='{{url "dynamic-subdomain4" "username4" "theParam1" "theparam2AfterStatic" "otherParam" "matchAnything"}}') username4.127.0.0.1:8080/mypath4/:param1/statichere/:param2/:otherparam/*something

p.
 a(href='{{url "dynamic-subdomain5" .ParamsAsArray }}') username5.127.0.0.1:8080/mypath6/:param1/:param2/staticParam/:param3AfterStatic


```

```go
// ./main.go
// Package main same example as template_html_5 but for pug/jade
package main

import (
	"github.com/kataras/go-template/pug"
	"github.com/kataras/iris"
)

func main() {
	iris.UseTemplate(pug.New()).Directory("./templates", ".jade")

	wildcard := iris.Party("*.")
	{
		wildcard.Get("/mypath", emptyHandler)("dynamic-subdomain1")
		wildcard.Get("/mypath2/:param1/:param2", emptyHandler)("dynamic-subdomain2")
		wildcard.Get("/mypath3/:param1/statichere/:param2", emptyHandler)("dynamic-subdomain3")
		wildcard.Get("/mypath4/:param1/statichere/:param2/:otherparam/*something", emptyHandler)("dynamic-subdomain4")
		wildcard.Get("/mypath5/:param1/:param2/staticParam/:param3AfterStatic", emptyHandler)("dynamic-subdomain5")
	}

	iris.Get("/", func(ctx *iris.Context) {
		// for dynamic_subdomain:8080/mypath5...
		// the first parameter is always the subdomain part
		paramsAsArray := []string{"username5", "theParam1", "theParam2", "theParam3"}

		if err := ctx.Render("page.jade", iris.Map{"ParamsAsArray": paramsAsArray}); err != nil {
			panic(err)
		}
	})

	iris.Get("/redirect/:namedRoute/:subdomain", func(ctx *iris.Context) {
		routeName := ctx.Param("namedRoute")
		subdomain := ctx.Param("subdomain")
		println("The full uri of " + routeName + "is: " + iris.URL(routeName, subdomain))
		// if routeName == "dynamic-subdomain1" && subdomain == "username1"
		// prints: The full uri ofd ynamic-subdomain1 is: http://username1.127.0.0.1:8080/mypath
		ctx.RedirectTo(routeName, subdomain) // the second parameter is the arguments, the first argument for dynamic subdomains is the subdomain part, after this, the named parameters
		// http://127.0.0.1:8080/redirect/my-subdomain1 will redirect to ->  http://username1.127.0.0.1:8080/mypath
	})

	iris.Listen("127.0.0.1:8080")
}

func emptyHandler(ctx *iris.Context) {
	ctx.Write("[SUBDOMAIN: %s]Hello from Path: %s.", ctx.Subdomain(), ctx.PathString())
}

// Note than you can see more Pug/Jade syntax examples by navigating to https://github.com/Joker/jade


```

>  Note than you can see more Pug/Jade syntax examples by navigating [here](https://github.com/Joker/jade)

```html
<!-- ./templates/basic.amber -->
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


```go
// ./main.go
package main

import (
	"github.com/kataras/go-template/amber"
	"github.com/kataras/iris"
)

type mypage struct {
	Name string
}

func main() {

	iris.UseTemplate(amber.New()).Directory("./templates", ".amber")

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Render("basic.amber", mypage{"iris"}, iris.RenderOptions{"gzip": true})
	})

	iris.Listen(":8080")
}


```


**Custom template engine** 

Simply, you have to implement only **3  functions**, for load and execute the templates. One optionally (**Funcs() map[string]interface{}**) which is used to register the iris' helpers funcs like `{{ url }}` and `{{ urlpath }}`.

```go

type (
	// TemplateEngine the interface that all template engines must implement
	TemplateEngine interface {
		// LoadDirectory builds the templates, usually by directory and extension but these are engine's decisions
		LoadDirectory(directory string, extension string) error
		// LoadAssets loads the templates by binary
		// assetFn is a func which returns bytes, use it to load the templates by binary
		// namesFn returns the template filenames
		LoadAssets(virtualDirectory string, virtualExtension string, assetFn func(name string) ([]byte, error), namesFn func() []string) error

		// ExecuteWriter finds, execute a template and write its result to the out writer
		// options are the optional runtime options can be passed by user
		// an example of this is the "layout" or "gzip" option
		ExecuteWriter(out io.Writer, name string, binding interface{}, options ...map[string]interface{}) error
	}

	// TemplateEngineFuncs is optional interface for the TemplateEngine
	// used to insert the Iris' standard funcs, see var 'usedFuncs'
	TemplateEngineFuncs interface {
		// Funcs should returns the context or the funcs,
		// this property is used in order to register the iris' helper funcs
		Funcs() map[string]interface{}
	}
)

```

The simplest implementation, which you can look as example, is the Markdown Engine, which is located [here](https://github.com/kataras/go-template/tree/master/markdown/markdown.go).



**iris.TemplateString**


Executes and parses the template but instead of rendering to the client, it returns the contents. Useful when you want to send a template via e-mail or anything you can imagine.

```html
<!-- ./templates/mypage.html -->
<html>
<head>
<title>Hello Django from Iris</title>

</head>
<body>
	{% if is_admin %}
	<p>{{username}} is an admin!</p>
	{% endif %}
</body>
</html>

```


```go
// ./main.go
package main

import (
	"github.com/kataras/go-template/django"
	"github.com/kataras/iris"
)

func main() {

	iris.UseTemplate(django.New()).Directory("./templates", ".html")

	iris.Get("/", func(ctx *iris.Context) {
		// THIS WORKS WITH ALL TEMPLATE ENGINES, but I am not doing the same example for all engines again :) (the same you can do with templates using the iris.ResponseString)
		rawHtmlContents := iris.TemplateString("mypage.html", map[string]interface{}{"username": "iris", "is_admin": true}, iris.RenderOptions{"charset": "UTF-8"}) // defaults to UTF-8 already
		ctx.Log(rawHtmlContents)
		ctx.Write("The Raw HTML is:\n%s", rawHtmlContents)
	})

	iris.Listen(":8080")
}


```

 > Note that: iris.TemplateString can be called outside of the context also




-----


 - examples are located [here](https://github.com/iris-contrib/examples/tree/master/template_engines/) 

- You can contribute to create more template engines for Iris, click [here](https://github.com/kataras/go-template) to navigate to the repository. 