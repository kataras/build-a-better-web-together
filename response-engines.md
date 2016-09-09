## Install

Install one response engine and all will be installed.

```sh
$ go get -u github.com/iris-contrib/response/json
```

## Iris' Station configuration

Remember, by 'station' we mean the default `iris.$CALL ` or `api:= iris.New(); api.$CALL` 

```go
iris.Config.Gzip = true // compresses/gzips response content to the client (same for Template Engines), defaults to false
iris.Config.Charset = "UTF-8" // defaults to "UTF-8" (same for Template Engines also)

// or
iris.Set(iris.OptionGzip(true), iris.OptionCharset("UTF-8"))
// or
iris.New(iris.OptionGzip(true), iris.OptionCharset("UTF-8"))
// or 
iris.New(iris.Configuration{ Gzip:true, Charset: "UTF-8" })
```

They can be overriden for specific `Render` actions: 
```go
func(ctx *iris.Context){
 ctx.Render("any/contentType", anyValue{}, iris.RenderOptions{"gzip":false, "charset": "UTF-8"})
}
```

## How to use

First of all don't be scared about the 'big' article, a response engine is very simple and is easy to understand.
Let's see what built-in response types are available in `iris.Context`.


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

**Text Response Engine**

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
		// logs if any error happens and sends a http status '500 internal server error' to the client
		ctx.MustRender("text/plain", myString)
	})

	iris.Listen(":8080")
}
```

**Custom response engine**

You can create a custom response engine using a func or an interface which implements the
` iris.ResponseEngine`  which contains a simple function: ` Response(val interface{}, options ...map[string]interface{}) ([]byte, error)
` 

A custom engine can be used to register a totally new content writer for a known ContentType or for a custom ContentType.  

You can imagine its useful, I will show you one right now.

Let's do a 'trick' here, which works for all other response engines, custom or not:
say for example, that you want a static'footer/suffix' on your content.

If a response engine has the same key and the same content type then the contents are appended and the final result will be rendered to the client
.

Let's do this with the `text/plain` content type, because you can see its results easly.
The first engine will use this  as key, the second & third will use the same key the first ("text/plain" is also the ContentType).
```go
package main

import (
	"github.com/iris-contrib/response/text"
	"github.com/kataras/iris"
)

func main() {
	// here we are registering the default text/plain, and after that we register the 'appender' only.
	// we have to register the default because we will add more response engines with the same content,
	// iris will not register this by-default if other response engines with the same ContentType already exist
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

// This is a way on how you can create a raw iris.ResponseEngine implementation:

// CustomTextEngine the response engine which appends a simple string on the default's text engine
type CustomTextEngine struct{}

// Implement the iris.ResponseEngine
func (e *CustomTextEngine) Response(val interface{}, options ...map[string]interface{}) ([]byte, error) {
	// we don't need the val, because we want only to append, so what we should do?
	// just return the []byte we want to be appended after the first registered text/plain engine

	return []byte("\nThis is the static FIRST suffix!"), nil
}

```

**iris.ResponseString**


ResponseString gives you the result of the response engine's work, it doesn't renders to the client but you can use
this function to collect the end result and send it via e-mail to the user, or anything you can imagine.


```go
package main

import "github.com/kataras/iris"

func main() {
	markdownContents := `## Hello Markdown from Iris

This is an example of Markdown with Iris



Features
--------

All features of Sundown are supported, including:

*   **Compatibility**. The Markdown v1.0.3 test suite passes with
    the --tidy option.  Without --tidy, the differences are
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
	
	[this is a link](https://github.com/kataras/iris) `

	iris.Get("/", func(ctx *iris.Context) {
		// let's see
		// convert markdown string to html and print it to the logger
		// THIS WORKS WITH ALL RESPONSE ENGINES, but I am not doing the same example for all engines again :) (the same you can do with templates using the iris.TemplateString)
		htmlContents := iris.ResponseString("text/markdown", markdownContents, iris.RenderOptions{"charset": "8859-1"}) // default is the iris.Config.Charset, which is UTF-8

		ctx.Log(htmlContents)
		ctx.Write("The Raw HTML is:\n%s", htmlContents)
	})

	iris.Listen(":8080")
}
```


Now we can continue to the rest of the default & built'n response engines


**JSON Response Engine**


```go
package main

import "github.com/kataras/iris"

type myjson struct {
	Name string `json:"name"`
}

func main() {

	iris.Get("/", func(ctx *iris.Context) {
		ctx.JSON(iris.StatusOK, iris.Map{"name": "iris"})
	})

	iris.Get("/alternative_1", func(ctx *iris.Context) {
		ctx.JSON(iris.StatusOK, myjson{Name: "iris"})
	})

	iris.Get("/alternative_2", func(ctx *iris.Context) {
		ctx.Render("application/json", myjson{Name: "iris"})
	})

	iris.Get("/alternative_3", func(ctx *iris.Context) {
		ctx.RenderWithStatus(iris.StatusOK, "application/json", myjson{Name: "iris"})
	})

	iris.Get("/alternative_4", func(ctx *iris.Context) {
		ctx.Render("application/json", myjson{Name: "iris"}, iris.RenderOptions{"charset": "UTF-8"}) // UTF-8 is the default.
	})

	iris.Get("/alternative_5", func(ctx *iris.Context) {
		// logs if any error and sends http status '500 internal server error' to the client
		ctx.MustRender("application/json", myjson{Name: "iris"}, iris.RenderOptions{"charset": "UTF-8"}) // UTF-8 is the default.
	})

	iris.Listen(":8080")
}

```


```go
package main

import (
	"github.com/iris-contrib/response/json"
	"github.com/kataras/iris"
)

type myjson struct {
	Name string `json:"name"`
}

func main() {
	iris.Config.Charset = "UTF-8" // this is the default, which you can change

	// first example
	// use the json's Config, we need the import of the json response engine in order to change its internal configs
	// this is one of the reasons you need to import a default engine,(template engine or response engine)
	/*
		type Config struct {
			Indent        bool
			UnEscapeHTML  bool
			Prefix        []byte
			StreamingJSON bool
		}
	*/
	iris.UseResponse(json.New(json.Config{
		Prefix: []byte("MYPREFIX"),
	}), json.ContentType) // you can use anything as the second parameter, the json.ContentType is the string "application/json", the context.JSON renders with this engine's key.

	jsonHandlerSimple := func(ctx *iris.Context) {
		ctx.JSON(iris.StatusOK, myjson{Name: "iris"})
	}

	jsonHandlerWithRender := func(ctx *iris.Context) {
		// you can also change the charset for a specific render action with RenderOptions
		ctx.Render("application/json", myjson{Name: "iris"}, iris.RenderOptions{"charset": "8859-1"})
	}

	// second example,
	// imagine that we need the context.JSON to be listening to our "application/json" response engine with a custom prefix (we did that before)
	// but we also want a different renderer, but again application/json content type, with indent option set to true:
	iris.UseResponse(json.New(json.Config{Indent: true}), "json2")("application/json")
	// yes the UseResponse returns a function which you can map the content type if it's not declared on the key
	json2Handler := func(ctx *iris.Context) {
		ctx.Render("json2", myjson{Name: "My iris"})
	}

	iris.Get("/", jsonHandlerSimple)

	iris.Get("/render", jsonHandlerWithRender)

	iris.Get("/json2", json2Handler)

	iris.Listen(":8080")
}

```


**JSONP Response Engine**

```go
package main

import "github.com/kataras/iris"

type myjson struct {
	Name string `json:"name"`
}

func main() {

	iris.Get("/", func(ctx *iris.Context) {
		ctx.JSONP(iris.StatusOK, "callbackName", iris.Map{"name": "iris"})
	})

	iris.Get("/alternative_1", func(ctx *iris.Context) {
		ctx.JSONP(iris.StatusOK, "callbackName", myjson{Name: "iris"})
	})

	iris.Get("/alternative_2", func(ctx *iris.Context) {
		ctx.Render("application/javascript", myjson{Name: "iris"}, iris.RenderOptions{"callback": "callbackName"})
	})

	iris.Get("/alternative_3", func(ctx *iris.Context) {
		ctx.RenderWithStatus(iris.StatusOK, "application/javascript", myjson{Name: "iris"}, iris.RenderOptions{"callback": "callbackName"})
	})

	iris.Get("/alternative_4", func(ctx *iris.Context) {
		// logs if any error happens and sends the http status '500 internal server error' to the client
		ctx.MustRender("application/javascript", myjson{Name: "iris"}, iris.RenderOptions{"callback": "callbackName", "charset": "UTF-8"}) // UTF-8 is the default.
	})

	iris.Listen(":8080")
}

```


```go
package main

import (
	"github.com/iris-contrib/response/jsonp"
	"github.com/kataras/iris"
)

type myjson struct {
	Name string `json:"name"`
}

func main() {
	iris.Config.Charset = "UTF-8" // this is the default, which you can change

	// first example
	// this is one of the reasons you need to import a default engine,(template engine or response engine)
	/*
		type Config struct {
			Indent   bool
			Callback string // the callback can be override by the context's options or parameter on context.JSONP
		}
	*/
	iris.UseResponse(jsonp.New(jsonp.Config{Indent: true}), jsonp.ContentType)
	// you can use anything as the second parameter,
	// the jsonp.ContentType is the string "application/javascript",
	// the context.JSONP renders with this engine's key.

	handlerSimple := func(ctx *iris.Context) {
		ctx.JSONP(iris.StatusOK, "callbackName", myjson{Name: "iris"})
	}

	handlerWithRender := func(ctx *iris.Context) {
		// you can also change the charset for a specific render action with RenderOptions
		ctx.Render("application/javascript", myjson{Name: "iris"}, iris.RenderOptions{"callback": "callbackName", "charset": "8859-1"})
	}

	// second example,
	// but we also want a different renderer, but again "application/javascript" as content type, with Callback option setted globaly:
	iris.UseResponse(jsonp.New(jsonp.Config{Callback: "callbackName"}), "jsonp2")("application/javascript")
	// yes the UseResponse returns a function which you can map the content type with if it's not declared on the key
	handlerJsonp2 := func(ctx *iris.Context) {
		ctx.Render("jsonp2", myjson{Name: "My iris"})
	}

	iris.Get("/", handlerSimple)

	iris.Get("/render", handlerWithRender)

	iris.Get("/jsonp2", handlerJsonp2)

	iris.Listen(":8080")
}
```



**XML Response Engine**


```go
package main

import "github.com/kataras/iris"

type myxml struct {
	XMLName xml.Name `xml:"xml_example"`
	First   string   `xml:"first,attr"`
	Second  string   `xml:"second,attr"`
}

func main() {

	iris.Get("/", func(ctx *iris.Context) {
		ctx.XML(iris.StatusOK, iris.Map{"first": "first attr ", "second": "second attr"})
	})

	iris.Get("/alternative_1", func(ctx *iris.Context) {
		ctx.XML(iris.StatusOK, myxml{First: "first attr", Second: "second attr"})
	})

	iris.Get("/alternative_2", func(ctx *iris.Context) {
		ctx.Render("text/xml", myxml{First: "first attr", Second: "second attr"})
	})

	iris.Get("/alternative_3", func(ctx *iris.Context) {
		ctx.RenderWithStatus(iris.StatusOK, "text/xml", myxml{First: "first attr", Second: "second attr"})
	})

	iris.Get("/alternative_4", func(ctx *iris.Context) {
		ctx.Render("text/xml", myxml{First: "first attr", Second: "second attr"}, iris.RenderOptions{"charset": "UTF-8"}) // UTF-8 is the default.
	})

	iris.Get("/alternative_5", func(ctx *iris.Context) {
		// logs if any error happens and sends the http status '500 internal server error' to the client
		ctx.MustRender("text/xml", myxml{First: "first attr", Second: "second attr"}, iris.RenderOptions{"charset": "UTF-8"})
	})

	iris.Listen(":8080")
}

```


```go
package main

import (
	encodingXML "encoding/xml"

	"github.com/iris-contrib/response/xml"
	"github.com/kataras/iris"
)

type myxml struct {
	XMLName encodingXML.Name `xml:"xml_example"`
	First   string           `xml:"first,attr"`
	Second  string           `xml:"second,attr"`
}

func main() {
	iris.Config.Charset = "UTF-8" // this is the default, which you can change

	// first example
	// this is one of the reasons you need to import a default engine,(template engine or response engine)
	/*
		type Config struct {
			Indent bool
			Prefix []byte
		}
	*/
	iris.UseResponse(xml.New(xml.Config{Indent: true}), xml.ContentType)
	// you can use anything as the second parameter,
	// the jsonp.ContentType is the string "text/xml",
	// the context.XML renders with this engine's key.

	handlerSimple := func(ctx *iris.Context) {
		ctx.XML(iris.StatusOK, myxml{First: "first attr", Second: "second attr"})
	}

	handlerWithRender := func(ctx *iris.Context) {
		// you can also change the charset for a specific render action with RenderOptions
		ctx.Render("text/xml", myxml{First: "first attr", Second: "second attr"}, iris.RenderOptions{"charset": "8859-1"})
	}

	// second example,
	// but we also want a different renderer, but again "text/xml" as content type, with prefix option set by configuration:
	iris.UseResponse(xml.New(xml.Config{Prefix: []byte("")}), "xml2")("text/xml") // if you really use a PREFIX it will not be valid xml, use it only for special cases
	// yes the UseResponse returns a function which you can map the content type with if it's not declared on the key
	handlerXML2 := func(ctx *iris.Context) {
		ctx.Render("xml2", myxml{First: "first attr", Second: "second attr"})
	}

	iris.Get("/", handlerSimple)

	iris.Get("/render", handlerWithRender)

	iris.Get("/xml2", handlerXML2)

	iris.Listen(":8080")
}
```


**Markdown Response Engine**


```go
package main

import "github.com/kataras/iris"

type myjson struct {
	Name string `json:"name"`
}

func main() {
	markdownContents := `## Hello Markdown from Iris

This is an example of Markdown with Iris



Features
--------

All features of Sundown are supported, including:

*   **Compatibility**. The Markdown v1.0.3 test suite passes with
    the --tidy option.  Without --tidy, the differences are
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
	
	[this is a link](https://github.com/kataras/iris) `

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Markdown(iris.StatusOK, markdownContents)
	})

	iris.Get("/alternative_1", func(ctx *iris.Context) {
		htmlContents := ctx.MarkdownString(markdownContents)
		ctx.HTML(iris.StatusOK, htmlContents)
	})

	// text/markdown is just the key which the markdown response engine and ctx.Markdown communicate,
	// it's real content type is text/html
	iris.Get("/alternative_2", func(ctx *iris.Context) {
		ctx.Render("text/markdown", markdownContents)
	})

	iris.Get("/alternative_3", func(ctx *iris.Context) {
		ctx.RenderWithStatus(iris.StatusOK, "text/markdown", markdownContents)
	})

	iris.Get("/alternative_4", func(ctx *iris.Context) {
		ctx.Render("text/markdown", markdownContents, iris.RenderOptions{"charset": "UTF-8"}) // UTF-8 is the default.
	})

	iris.Get("/alternative_5", func(ctx *iris.Context) {
		// logs if any error happens and sends the http status '500 internal server error' to the client
		ctx.MustRender("text/markdown", markdownContents, iris.RenderOptions{"charset": "UTF-8"}) // UTF-8 is the default.
	})

	iris.Listen(":8080")
}

```


```go
package main

import (
	"github.com/iris-contrib/response/markdown"
	"github.com/kataras/iris"
)

func main() {
	markdownContents := `## Hello Markdown from Iris

This is an example of Markdown with Iris



Features
--------

All features of Sundown are supported, including:

*   **Compatibility**. The Markdown v1.0.3 test suite passes with
    the --tidy option.  Without --tidy, the differences are
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
	
	[this is a link](https://github.com/kataras/iris) `

	//first example
	// this is one of the reasons you need to import a default engine,(template engine or response engine)
	/*
		type Config struct {
			MarkdownSanitize bool
		}
	*/
	iris.UseResponse(markdown.New(), markdown.ContentType)
	// you can use anything as the second parameter,
	// the markdown.ContentType is the string "text/markdown",
	// the context.Markdown renders with this engine's key.

	handlerWithRender := func(ctx *iris.Context) {
		// you can also change the charset for a specific render action with RenderOptions
		ctx.Render("text/markdown", markdownContents, iris.RenderOptions{"charset": "8859-1"})
	}

	//second example,
	// but we also want a different renderer, but again "text/markdown" as 'content type' (this is converted to text/html behind the scenes), with MarkdownSanitize option setted to true:
	iris.UseResponse(markdown.New(markdown.Config{MarkdownSanitize: true}), "markdown2")("text/markdown")
	// yes the UseResponse returns a function which you can map the content type if it's not declared on the key
	handlerMarkdown2 := func(ctx *iris.Context) {
		ctx.Render("markdown2", markdownContents, iris.RenderOptions{"gzip": true})
	}

	iris.Get("/", handlerWithRender)

	iris.Get("/markdown2", handlerMarkdown2)

	iris.Listen(":8080")
}


```

**Data(Binary) Response Engine**


```go
package main

import "github.com/kataras/iris"

func main() {
	myData := []byte("some binary data or a program here which will not be a simple string at the production")

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Data(iris.StatusOK, myData)
	})

	iris.Get("/alternative_1", func(ctx *iris.Context) {
		ctx.Render("application/octet-stream", myData)
	})

	iris.Get("/alternative_2", func(ctx *iris.Context) {
		ctx.RenderWithStatus(iris.StatusOK, "application/octet-stream", myData)
	})

	iris.Get("/alternative_3", func(ctx *iris.Context) {
		ctx.Render("application/octet-stream", myData, iris.RenderOptions{"gzip": true}) // gzip is false by default
	})

	iris.Get("/alternative_4", func(ctx *iris.Context) {
		// logs if any error and sends http status '500 internal server error' to the client
		ctx.MustRender("application/octet-stream", myData)
	})

	iris.Listen(":8080")
}

```


 ----- 

 - examples are located [here](https://github.com/iris-contrib/examples/tree/master/response_engines/).

- You can contribute response engines to Iris, click [here](https://github.com/iris-contrib/response) to navigate to the reository.
