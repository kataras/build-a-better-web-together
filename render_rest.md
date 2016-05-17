# REST

Provides functionality for easily rendering JSON, XML, Text, binary data and Markdown.

### config.Rest


```go
    // Appends the given character set to the Content-Type header. Default is "UTF-8".
	Charset string
	// Gzip enable it if you want to render with gzip compression. Default is false
	Gzip bool
	// Outputs human readable JSON.
	IndentJSON bool
	// Outputs human readable XML. Default is false.
	IndentXML bool
	// Prefixes the JSON output with the given bytes. Default is false.
	PrefixJSON []byte
	// Prefixes the XML output with the given bytes.
	PrefixXML []byte
	// Unescape HTML characters "&<>" to their original values. Default is false.
	UnEscapeHTML bool
	// Streams JSON responses instead of marshalling prior to sending. Default is false.
	StreamingJSON bool
	// Disables automatic rendering of http.StatusInternalServerError
    // when an error occurs. Default is false.
	DisableHTTPErrorRendering bool
    // MarkdownSanitize sanitizes the markdown. Default is false.
    MarkdownSanitize bool

```
```go
//...
import (
  "github.com/kataras/iris"
  "github.com/kataras/iris/config"
)
//...

//1.
iris.Config().Render.Rest.IndentJSON = true
iris.Config().Render.Rest...
//2.
restConfig:= config.Rest{
	Charset:                   "UTF-8",
	IndentJSON:                false,
	IndentXML:                 false,
	PrefixJSON:                []byte(""),
	PrefixXML:                 []byte(""),
	UnEscapeHTML:              false,
	StreamingJSON:             false,
	DisableHTTPErrorRendering: false,
}

iris.Config().Rest = restConfig


```
### Usage
The rendering functions simply wraps Go's existing functionality for marshaling and rendering data.

- JSON: Uses the [encoding/json](http://golang.org/pkg/encoding/json/) package to marshal data into a JSON-encoded response.
- XML: Uses the [encoding/xml](http://golang.org/pkg/encoding/xml/) package to marshal data into an XML-encoded response.
- Binary data: Passes the incoming data straight through to the `iris.Context.Response`.
- Text: Passes the incoming string straight through to the ``iris.Context.Response``.


~~~ go
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
          ctx.JSON(iris.StatusOK, map[string]string{"hello": "json"})
      })

      iris.Get("/jsonp", func(ctx *iris.Context) {
          ctx.JSONP(iris.StatusOK, "callbackName", map[string]string{"hello": "jsonp"})
      })

      iris.Get("/xml", func(ctx *iris.Context) {
          ctx.XML(iris.StatusOK, ExampleXml{One: "hello", Two: "xml"})
      })

      iris.Listen(":8080")
  }
~~~