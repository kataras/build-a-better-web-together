# Content Negotiation

**Sometimes a server application needs to serve different representations of a resource at the same URI**. Of course this can be done by hand, manually checking the `Accept` request header and push the requested form of the content. However, as your app manages more resources and different kind of representations this can be very painful, as you may need to check for `Accept-Charset`, `Accept-Encoding`, put some server-side priorities , handle the errors correctly and e.t.c.

There are some web frameworks in Go already struggle to implement a feature like this but they don't do it correctly:

- they don't handle accept-charset at all
- they don't handle accept-encoding at all
- they don't send error status code (406 not acceptable) as RFC proposes and more...

But, fortunately for us, **Iris always follows the best practises and the Web standards**.

Based on:
- https://developer.mozilla.org/en-US/docs/Web/HTTP/Content_negotiation
- https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept
- https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Charset
- https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Encoding

Implemented on:
- https://github.com/kataras/iris/pull/1316/commits/8ee0de51c593fe0483fbea38117c3c88e065f2ef

---------

## Example

```go
type testdata struct {
	Name string `json:"name" xml:"Name"`
	Age  int    `json:"age" xml:"Age"`
}
```

Render a resource with "gzip" encoding algorithm
as application/json or text/xml or application/xml

- when client's accept header contains one of them
- or JSON (the first declared) if accept is empty,
- and when client's accept-encoding header contains "gzip" or it's empty.

```go
app.Get("/resource", func(ctx iris.Context) {
	data := testdata{
		Name: "test name",
		Age:  26,
	}

        ctx.Negotiation().JSON().XML().EncodingGzip()

	_, err := ctx.Negotiate(data)
	if err != nil {
		ctx.Writef("%v", err)
	}
})

```

**OR** define them in a middleware and call Negotiate with nil in the final handler.

```go
ctx.Negotiation().JSON(data).XML(data).Any("content for */*")
ctx.Negotiate(nil)
```

```go
app.Get("/resource2", func(ctx iris.Context) {
	jsonAndXML := testdata{
		Name: "test name",
		Age:  26,
	}

	ctx.Negotiation().
		JSON(jsonAndXML).
		XML(jsonAndXML).
		HTML("<h1>Test Name</h1><h2>Age 26</h2>")

	ctx.Negotiate(nil)
})
```

[Read the full example](https://github.com/kataras/iris/blob/8ee0de51c593fe0483fbea38117c3c88e065f2ef/_examples/http_responsewriter/content-negotiation/main.go#L22).


## Documentation

The [Context.Negotiation](https://github.com/kataras/iris/blob/8ee0de51c593fe0483fbea38117c3c88e065f2ef/context/context.go#L3342) method creates once and returns the negotiation builder
to build server-side available prioritized content for specific content type(s), charset(s) and encoding algorithm(s).

```go
Context.Negotiation() *context.NegotiationBuilder
```

The [Context.Negotiate](https://github.com/kataras/iris/blob/8ee0de51c593fe0483fbea38117c3c88e065f2ef/context/context.go#L3402) method used for serving different representations of a resource at the same URI. It returns `context.ErrContentNotSupported` when not matched mime type(s).

- The "v" can be a single [iris.N](https://github.com/kataras/iris/blob/8ee0de51c593fe0483fbea38117c3c88e065f2ef/context/context.go#L3298-L3309) struct value.
- The "v" can be any value completes the [context.ContentSelector](https://github.com/kataras/iris/blob/8ee0de51c593fe0483fbea38117c3c88e065f2ef/context/context.go#L3272) interface.
- The "v" can be any value completes the [context.ContentNegotiator](https://github.com/kataras/iris/blob/8ee0de51c593fe0483fbea38117c3c88e065f2ef/context/context.go#L3281) interface.
- The "v" can be any value of struct(JSON, JSONP, XML, YAML) or
string(TEXT, HTML) or []byte(Markdown, Binary) or []byte with any matched mime type.
- If the "v" is nil, the `Context.Negotitation()` builder's
content will be used instead, otherwise "v" overrides builder's content
(server mime types are still retrieved by its registered, supported, mime list)

- Set mime type priorities by [Negotiation().MIME.Text.JSON.XML.HTML...](https://github.com/kataras/iris/blob/8ee0de51c593fe0483fbea38117c3c88e065f2ef/context/context.go#L3500-L3621).
- Set charset priorities by [Negotiation().Charset(...)](https://github.com/kataras/iris/blob/8ee0de51c593fe0483fbea38117c3c88e065f2ef/context/context.go#L3640).
- Set encoding algorithm priorities by [Negotiation().Encoding(...)](https://github.com/kataras/iris/blob/8ee0de51c593fe0483fbea38117c3c88e065f2ef/context/context.go#L3652-L3665).
- Modify the accepted by [Negotiation().Accept./Override()/.XML().JSON().Charset(...).Encoding(...)...](https://github.com/kataras/iris/blob/8ee0de51c593fe0483fbea38117c3c88e065f2ef/context/context.go#L3774-L3877).

```go
Context.Negotiate(v interface{}) (int, error)
```
