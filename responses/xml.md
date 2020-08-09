# XML

**Content-Type: "text/xml"**

Extensible Markup Language (**XML**) is a markup language that defines a set of rules for encoding documents in a format that is both human-readable and machine-readable.

The `Context.XML(v, ...opts)` is the method which sends XML responses to the client. It accepts the value and optional settings for rendering. The `XML` options structure looks like this:

```go
type XML struct {
	Indent string
	Prefix string
}
```

> If `Indent` field is empty and the application runs without optimizations, the `Indent` field will be automatically set to `2 spaces`.

Render a Go struct as XML response:

```go
type ExampleXML struct {
	XMLName xml.Name `xml:"example"`
	One     string   `xml:"one,attr"`
	Two     string   `xml:"two,attr"`
}

func handler(ctx iris.Context) {
    response := ExampleXML{One: "hello", Two: "xml"}
    ctx.XML(response)
}
```

**Result**

```xml
<example one="hello" two="xml"/>
```

You can **NOT** provide a raw `map` or `iris.Map` on `context.XML` because the value should complete the [xml.Marshaler](https://godoc.org/encoding/xml#Marshaler) interface. Iris has the `iris.XMLMap` which converts a `map` (or `iris.Map`) to xml Marshaler. That helper function accepts the root level name and a map. Example:

```go
func handler(ctx iris.Context) {
    kv := iris.Map{"key": "value"}
    response := iris.XMLMap("keys", kv)
    ctx.XML(response, iris.XML{Indent: "    "})
}
```

**Result**

```xml
<keys>
    <key>value</key>
</keys>
```

References:
- [Specification](https://www.w3.org/TR/REC-xml/)
- [SOAP vs REST vs JSON - a 2020 comparison](https://raygun.com/blog/soap-vs-rest-vs-json/)

<!-- slide:break-100 -->
