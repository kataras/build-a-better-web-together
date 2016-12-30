## Install

Default Serializers[*](https://github.com/kataras/go-serializers/) are already installed when Iris has been installed.   

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

First of all don't be scared about the 'big' article, a serialize engine(serializer, old: Serializer) is very simple and is easy to understand.
Let's see what built-in response types are available in `iris.Context`.


```go
package main

import (
	"encoding/xml"

	"gopkg.in/kataras/iris.v5"
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

**Text Serializer**

```go
package main

import "gopkg.in/kataras/iris.v5"

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

**Custom Serializer**

You can create a custom Serializer using a func or an interface which implements the
` serializer.Serializer`  which contains a simple function: ` Serialize(val interface{}, options ...map[string]interface{}) ([]byte, error)
` 

A custom engine can be used to register a totally new content writer for a known ContentType or for a custom ContentType.  

You can imagine its useful, I will show you one right now.

Let's do a 'trick' here, which works for all other Serializers, custom or not:
say for example, that you want a static'footer/suffix' on your content.

If a Serializer has the same key and the same content type then the contents are appended and the final result will be rendered to the client
.

Let's do this with the `text/plain` content type, because you can see its results easly.
```go
// You can create a custom serialize engine(serializer) using a func or an interface which implements the
// serializer.Serializer which contains a simple function: Serialize(val interface{}, options ...map[string]interface{}) ([]byte, error)

// A custom engine can be used to register a totally new content writer for a known ContentType or for a custom ContentType

// Let's do a 'trick' here, which works for all other serialize engine(serializer)s, custom or not:

// say for example, that you want a static'footer/suffix' on your content, without the need to create & register a middleware for that, per route or globally
// you want to be even more organised.
//
// IF a serialize engine(serializer) has the same key and the same content type then the contents are appended and the final result will be rendered to the client.

// Enough with my 'bad' english, let's code something small:

package main

import (
	"github.com/kataras/go-serializer"
	"github.com/kataras/go-serializer/text"
	"gopkg.in/kataras/iris.v5"
)

// Let's do this with ` text/plain` content type, because you can see its results easly, the first engine will use this "text/plain" as key,
// the second & third will use the same, as firsts, key, which is the ContentType also.
func main() {
	// we are registering the default text/plain,  and after we will register the 'appender' only
	// we have to register the default because we will add more serialize engine(serializer)s with the same content,
	// iris will not register this by-default if other serialize engine(serializer) with the corresponding ContentType already exists
	iris.UseSerializer(text.ContentType, text.New())

	// register a serialize engine(serializer) serializer.Serializer
	iris.UseSerializer(text.ContentType, &CustomTextEngine{})
	// register a serialize engine(serializer) with func
	iris.UseSerializer(text.ContentType, serializer.SerializeFunc(func(val interface{}, options ...map[string]interface{}) ([]byte, error) {
		return []byte("\nThis is the static SECOND AND LAST suffix!"), nil
	}))

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Text(iris.StatusOK, "Hello!") // or ctx.Render(text.ContentType," Hello!")
	})

	iris.Listen(":8080")
}

// This is the way you create one with raw serialiser.Serializer implementation:

// CustomTextEngine the serialize engine(serializer) which appends a simple string on the default's text engine
type CustomTextEngine struct{}

// Implement the serializer.Serializer
func (e *CustomTextEngine) Serialize(val interface{}, options ...map[string]interface{}) ([]byte, error) {
	// we don't need the val, because we want only to append, so what we should do?
	// just return the []byte we want to be appended after the first registered text/plain engine
	return []byte("\nThis is the static FIRST suffix!"), nil
}


```

**iris.SerializeToString**


SerializeToString gives you the result of the Serializer's work, it doesn't renders to the client but you can use
this function to collect the end result and send it via e-mail to the user, or anything you can imagine.


```go
package main

import "gopkg.in/kataras/iris.v5"

func main() {

	// SerializeToString gives you the result of the serialize engine(serializer)'s work, it doesn't renders to the client but you can use
	// this function to collect the end result and send it via e-mail to the user, or anything you can imagine.

	// Note that: iris.SerializeToString is called outside of the context, using your iris $instance (iris. is the default)

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
		// THIS WORKS WITH ALL serialize engine(serializer)S, but I am not doing the same example for all engines again :) (the same you can do with templates using the iris.TemplateString)
		htmlContents := iris.SerializeToString("text/markdown", markdownContents, iris.RenderOptions{"charset": "8859-1"}) // default is the iris.Config.Charset, which is UTF-8

		ctx.Log(htmlContents)
		ctx.Write("The Raw HTML is:\n%s", htmlContents)
	})

	iris.Listen(":8080")
}

```


Now we can continue to the rest of the default & built'n Serializers


**JSON Serializer**


```go
package main

import "gopkg.in/kataras/iris.v5"

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
	"github.com/kataras/go-serializer/json"
	"gopkg.in/kataras/iris.v5"
)

type myjson struct {
	Name string `json:"name"`
}

func main() {
	iris.Config.Charset = "UTF-8" // this is the default, which you can change

	//first example
	// use the json's Config, we need the import of the json serialize engine(serializer) in order to change its internal configs
	// this is one of the reasons you need to import a default engine,(template engine or serialize engine(serializer))
	/*
		type Config struct {
			Indent        bool
			UnEscapeHTML  bool
			Prefix        []byte
			StreamingJSON bool
		}
	*/
	iris.UseSerializer(json.ContentType, json.New(json.Config{
		Prefix: []byte("MYPREFIX"),
	})) // you can use anything as the second parameter, the json.ContentType is the string "application/json", the context.JSON renders with this engine's key.

	jsonHandlerSimple := func(ctx *iris.Context) {
		ctx.JSON(iris.StatusOK, myjson{Name: "iris"})
	}

	jsonHandlerWithRender := func(ctx *iris.Context) {
		// you can also change the charset for a specific render action with RenderOptions
		ctx.Render("application/json", myjson{Name: "iris"}, iris.RenderOptions{"charset": "8859-1"})
	}

	//second example,
	// imagine that we need the context.JSON to be listening to our "application/json" serialize engine(serializer) with a custom prefix (we did that before)
	// but we also want a different renderer, but again application/json content type, with Indent option setted to true:
	iris.UseSerializer("json2", json.New(json.Config{Indent: true}))
	json2Handler := func(ctx *iris.Context) {
		ctx.Render("json2", myjson{Name: "My iris"})
		ctx.SetContentType("application/json")
	}

	iris.Get("/", jsonHandlerSimple)

	iris.Get("/render", jsonHandlerWithRender)

	iris.Get("/json2", json2Handler)

	iris.Listen(":8080")
}


```


**JSONP Serializer**

```go
package main

import "gopkg.in/kataras/iris.v5"

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
		// logs if any error and sends http status '500 internal server error' to the client
		ctx.MustRender("application/javascript", myjson{Name: "iris"}, iris.RenderOptions{"callback": "callbackName", "charset": "UTF-8"}) // UTF-8 is the default.
	})

	iris.Listen(":8080")
}


```


```go
package main

import (
	"github.com/kataras/go-serializer/jsonp"
	"gopkg.in/kataras/iris.v5"
)

type myjson struct {
	Name string `json:"name"`
}

func main() {
	iris.Config.Charset = "UTF-8" // this is the default, which you can change

	//first example
	// this is one of the reasons you need to import a default engine,(template engine or serialize engine(serializer))
	/*
		type Config struct {
			Indent   bool
			Callback string // the callback can be override by the context's options or parameter on context.JSONP
		}
	*/
	iris.UseSerializer(jsonp.ContentType, jsonp.New(jsonp.Config{
		Indent: true,
	}))
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

	//second example,
	// but we also want a different renderer, but again "application/javascript" as content type, with Callback option setted globaly:
	iris.UseSerializer("jsonp2", jsonp.New(jsonp.Config{Callback: "callbackName"}))
	// yes the UseSerializer returns a function which you can map the content type if it's not declared on the key
	handlerJsonp2 := func(ctx *iris.Context) {
		ctx.Render("jsonp2", myjson{Name: "My iris"})
		ctx.SetContentType("application/javascript")
	}

	iris.Get("/", handlerSimple)

	iris.Get("/render", handlerWithRender)

	iris.Get("/jsonp2", handlerJsonp2)

	iris.Listen(":8080")
}

```



**XML Serializer**


```go
package main

import (
	"encoding/xml"
	"gopkg.in/kataras/iris.v5"
)

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
		// logs if any error and sends http status '500 internal server error' to the client
		ctx.MustRender("text/xml", myxml{First: "first attr", Second: "second attr"}, iris.RenderOptions{"charset": "UTF-8"})
	})

	iris.Listen(":8080")
}


```


```go
package main

import (
	encodingXML "encoding/xml"

	"github.com/kataras/go-serializer/xml"
	"gopkg.in/kataras/iris.v5"
)

type myxml struct {
	XMLName encodingXML.Name `xml:"xml_example"`
	First   string           `xml:"first,attr"`
	Second  string           `xml:"second,attr"`
}

func main() {
	iris.Config.Charset = "UTF-8" // this is the default, which you can change

	//first example
	// this is one of the reasons you need to import a default engine,(template engine or serialize engine(serializer))
	/*
		type Config struct {
			Indent bool
			Prefix []byte
		}
	*/
	iris.UseSerializer(xml.ContentType, xml.New(xml.Config{
		Indent: true,
	}))
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

	//second example,
	// but we also want a different renderer, but again "text/xml" as content type, with prefix option setted by configuration:
	iris.UseSerializer("xml2", xml.New(xml.Config{Prefix: []byte("")})) // if you really use a PREFIX it will be not valid xml, use it only for special cases
	handlerXML2 := func(ctx *iris.Context) {
		ctx.Render("xml2", myxml{First: "first attr", Second: "second attr"})
		ctx.SetContentType("text/xml; charset=" + iris.Config.Charset)
	}

	iris.Get("/", handlerSimple)

	iris.Get("/render", handlerWithRender)

	iris.Get("/xml2", handlerXML2)

	iris.Listen(":8080")
}

```


**Markdown Serializer**


```go
package main

import "gopkg.in/kataras/iris.v5"

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

	// text/markdown is just the key which the markdown serialize engine(serializer) and ctx.Markdown communicate,
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
		// logs if any error and sends http status '500 internal server error' to the client
		ctx.MustRender("text/markdown", markdownContents, iris.RenderOptions{"charset": "UTF-8"}) // UTF-8 is the default.
	})

	iris.Listen(":8080")
}


```


```go
package main

import (
	"github.com/kataras/go-serializer/markdown"
	"gopkg.in/kataras/iris.v5"
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
	// this is one of the reasons you need to import a default engine,(template engine or serialize engine(serializer))
	/*
		type Config struct {
			MarkdownSanitize bool
		}
	*/
	iris.UseSerializer(markdown.ContentType, markdown.New())
	// you can use anything as the second parameter,
	// the markdown.ContentType is the string "text/markdown",
	// the context.Markdown renders with this engine's key.

	handlerWithRender := func(ctx *iris.Context) {
		// you can also change the charset for a specific render action with RenderOptions
		ctx.Render("text/markdown", markdownContents, iris.RenderOptions{"charset": "8859-1"})
	}

	//second example,
	// but we also want a different renderer, but again "text/html" as 'content type' (this is the real content type we want to render with, at the first ctx.Render the text/markdown key is converted automatically to text/html without need to call SetContentType), with MarkdownSanitize option setted to true:
	iris.UseSerializer("markdown2", markdown.New(markdown.Config{MarkdownSanitize: true}))
	handlerMarkdown2 := func(ctx *iris.Context) {
		ctx.Render("markdown2", markdownContents, iris.RenderOptions{"gzip": true})
		ctx.SetContentType("text/html")
	}

	iris.Get("/", handlerWithRender)

	iris.Get("/markdown2", handlerMarkdown2)

	iris.Listen(":8080")
}

```

**Data(Binary) Serializer**


```go
package main

import "gopkg.in/kataras/iris.v5"

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

 - examples are located [here](https://github.com/iris-contrib/examples/tree/5.0.0/serialize_engines/).

- You can contribute to Serializers, click [here](https://github.com/kataras/go-serializer) to navigate to the reository.
