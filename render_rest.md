# REST

[This is a package](https://github.com/kataras/iris/tree/master/rest)

Provides functionality for easily, one line, rendering JSON, XML, text and binary data.


## Usage
The rendering functions simply wraps Go's existing functionality for marshaling and rendering data.

- JSON: Uses the [encoding/json](http://golang.org/pkg/encoding/json/) package to marshal data into a JSON-encoded response.
- XML: Uses the [encoding/xml](http://golang.org/pkg/encoding/xml/) package to marshal data into an XML-encoded response.
- Binary data: Passes the incoming data straight through to the `iris.Context.Response`.
- Text: Passes the incoming string straight through to the ``iris.Context.Response``.
- 
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
~~~