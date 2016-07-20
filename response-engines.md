First of all don't be scary about the 'big' article here, a response engine works very simple and is easy to understand how.

Let's see what are the built'n response by content-type context's methods using the defaults only, unchanged, response engines.


```go

package main

import (
	"encoding/xml"

	"github.com/kataras/iris"
)

type ExampleXml struct {
	XMLName xml.Name `xml:"example"`
	One     string   `xml:"one,attr"`
	Two     string   `xml:"two,attr"`
}

func main() {
	iris.Get("/data", func(ctx *iris.Context) {
		ctx.Data(iris.StatusOK, []byte("Some binary data here."))
	})

	iris.Get("/text", func(ctx *iris.Context) {
		ctx.Text(iris.StatusOK, "Plain text here")
	})

	iris.Get("/json", func(ctx *iris.Context) {
		ctx.JSON(iris.StatusOK, map[string]string{"hello": "json"}) // or myjsonStruct{hello:"json}
	})

	iris.Get("/jsonp", func(ctx *iris.Context) {
		ctx.JSONP(iris.StatusOK, "callbackName", map[string]string{"hello": "jsonp"})
	})

	iris.Get("/xml", func(ctx *iris.Context) {
		ctx.XML(iris.StatusOK, ExampleXml{One: "hello", Two: "xml"}) // or iris.Map{"One":"hello"...}
	})

	iris.Get("/markdown", func(ctx *iris.Context) {
		ctx.Markdown(iris.StatusOK, "# Hello Dynamic Markdown Iris")
	})

	iris.Listen(":8080")
}


```


Bellow you will, propably, see how 'good' are my english (joke...), but at the end we're coders and some of us programmers too, so I hope you will be able to understand at least, the code snippets ( a lot of them, you will be tired from this simplicity ).




** Text Response Engine**

```go

package main

import "github.com/kataras/iris"

func main() {
	iris.Config.Charset = "UTF-8" // this is the default, you don't have to set it manually

	myString := "this is just a simple string which you can already render with ctx.Write"

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Text(iris.StatusOK, myString)
	})

	iris.Get("/alternative_1", func(ctx *iris.Context) {
		ctx.Render("text/plain", myString)
	})

	iris.Get("/alternative_2", func(ctx *iris.Context) {
		ctx.RenderWithStatus(iris.StatusOK, "text/plain", myString)
	})

	iris.Get("/alternative_3", func(ctx *iris.Context) {
		ctx.Render("text/plain", myString, iris.RenderOptions{"charset": "UTF-8"}) // default & global charset is UTF-8
	})

	iris.Get("/alternative_4", func(ctx *iris.Context) {
		// logs if any error and sends http status '500 internal server error' to the client
		ctx.MustRender("text/plain", myString)
	})

	iris.Listen(":8080")
}

```



You can create a custom response engine using a func or an interface which implements the
` iris.ResponseEngine`  which contains a simple function: ` Response(val interface{}, options ...map[string]interface{}) ([]byte, error)` 

A custom engine can be used to register a totally new content writer for a known ContentType or for a custom ContentType  

You can imagine its useful, I will show you one right now.

Let's do a 'trick' here, which works for all other response engines, custom or not:

say for example, that you want a static'footer/suffix' on your content.

IF a response engine has the same key and the same content type then the contents are appended and the final result will be rendered to the client.

Let's do this with ` text/plain` content type, because you can see its results easly, the first engine will use this "text/plain" as key also, the second will use other key but for the same ContentType.
```go

package main

import (
	"github.com/iris-contrib/response/text"
	"github.com/kataras/iris"
)

func main() {
	// here we are registering the default text/plain,  and after we will register the 'appender' only
	// we have to register the default because we will 
    // add more response engines with the same content,
	// iris will not register this by-default if 
    // other response engine with the corresponding ContentType already exists

	iris.UseResponse(text.New(), text.ContentType) // it's the key which happens to be a valid content-type also, "text/plain" so this will be used as the ContentType header

	// register a response engine: iris.ResponseEngine 
	iris.UseResponse(&CustomTextEngine{}, text.ContentType)
	
    // register a response engine with func
	iris.UseResponse(iris.ResponseEngineFunc(func(val interface{}, options ...map[string]interface{}) ([]byte, error) {
		return []byte("\nThis is the static SECOND AND LAST suffix!"), nil
	}), text.ContentType)

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Text(iris.StatusOK, "Hello!") // or ctx.Render(text.ContentType," Hello!")
	})

	iris.Listen(":8080")
}

// This is the way you create one with raw iris.ResponseEngine implementation:

// CustomTextEngine the response engine which appends a simple string on the default's text engine
type CustomTextEngine struct{}

// Implement the iris.ResponseEngine
func (e *CustomTextEngine) Response(val interface{}, options ...map[string]interface{}) ([]byte, error) {
	// we don't need the val, because we want only to append, so what to do?
	// just return the []byte we want to be appended after the first registered text/plain engine

	return []byte("\nThis is the static FIRST suffix!"), nil
}

```





