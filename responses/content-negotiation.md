# Content Negotiation

**Content-Type: _*_**

## What is it

**Sometimes a server application needs to serve different representations of a resource at the same URI**. Of course this can be done by hand, manually checking the `Accept` request header and push the requested form of the content. However, as your app manages more resources and different kind of representations this can be very painful, as you may need to check for `Accept-Charset`, `Accept-Encoding`, put some server-side priorities , handle the errors correctly and e.t.c.

There are some web frameworks in Go already struggle to implement a feature like this but they don't do it correctly:

* they don't handle accept-charset at all
* they don't handle accept-encoding at all
* they don't send error status code (406 not acceptable) as RFC proposes and more...

But, fortunately for us, **Iris always follows the best practises and the Web standards**.

Based on:

* [https://developer.mozilla.org/en-US/docs/Web/HTTP/Content\_negotiation](https://developer.mozilla.org/en-US/docs/Web/HTTP/Content_negotiation)
* [https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept)
* [https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Charset](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Charset)
* [https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Encoding](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Encoding)

Implemented on:

* [https://github.com/kataras/iris/pull/1316/commits/8ee0de51c593fe0483fbea38117c3c88e065f2ef\#diff-15cce7299aae8810bcab9b0bf9a2fdb1](https://github.com/kataras/iris/pull/1316/commits/8ee0de51c593fe0483fbea38117c3c88e065f2ef#diff-15cce7299aae8810bcab9b0bf9a2fdb1)

## Introduction

The Context method which is responsible to render specific mime types based on the client's `Accept` header is the `Negotiate` one.

Before `Negotiate` method fired, the handler MUST declare what mime types the server supports to render with safety. We can do that using the `Negotiation` priorities builder.

## The `Negotitation` priorities builder

The Context method which returns the builder is the `Negotiation` one.

```go
Negotiation() *NegotiationBuilder
```

It returns:

```go
type NegotiationBuilder struct {
    Accept NegotiationAcceptBuilder
    // [other unexproted fields]
}
```

The `Accept` struct **field** can be used to customize the **client's** Accept header manually, e.g. when the client does not contain an "application/json" mime type on its `Accept` header value.

Read more about [NegotiationAcceptBuilder](https://godoc.org/github.com/kataras/iris/context#NegotiationAcceptBuilder).

The `NegotitationBuilder` has the necessary **methods** to help you prioritize mime types, charsets and encoding. [Read the documentation](https://godoc.org/github.com/kataras/iris/context#NegotiationBuilder).

In short, it contains the following methods:

```go
// Declare custom mime and optionally content to render.
MIME(mime string, content interface{}) *NegotiationBuilder
// MIME method helpers:
Text(v ...string) *NegotiationBuilder
Markdown(v ...[]byte) *NegotiationBuilder
Binary(v ...[]byte) *NegotiationBuilder
JSON(v ...interface{}) *NegotiationBuilder
Problem(v ...interface{}) *NegotiationBuilder
JSONP(v ...interface{}) *NegotiationBuilder
XML(v ...interface{}) *NegotiationBuilder
YAML(v ...interface{}) *NegotiationBuilder
Protobuf(v ...interface{}) *NegotiationBuilder
MsgPack(v ...interface{}) *NegotiationBuilder
Any(v ...interface{}) *NegotiationBuilder
```

```go
// Charset
Charset(charset ...string) *NegotiationBuilder
```

```go
// Encoding (compression)
Encoding(encoding ...string) *NegotiationBuilder
EncodingGzip() *NegotiationBuilder
```

```go
// Clears all the above, resets the builder
// if necessary on another handler.
Clear() *NegotiationBuilder
```

The `Build` is called automatically on `Negotiate` method but
it it's exported for a custom implementation of negotiation by the end-developer if ever required.

```go
// Build calculates the client's and server's
// mime type(s), charset(s) and encoding
// and returns the final content type, charset and
// encoding that server should render to the client.
Build() (contentType, charset, encoding string, content interface{})
```

### Usage

The Negotiation is a context method which sets and returns the negotiation builder, so it can be used inside a middleware too.

To declare what mime types server can render and match versus the client's `Accept` header you can do that:

```go
func main() {
    users := app.Party("/users")
    users.Use(setAcceptTypes)
    // [...]
}

func setAcceptTypes(ctx iris.Context) {
    ctx.Negotiation().JSON().XML().HTML().EncodingGzip()
    ctx.Next()
}
```

The above will tell the server that we can accept JSON, XML and HTML mime types, optionally encoding by Gzip if the client supports. So far nothing is rendered to the client, learn about the `Negotiate` method below.

## The `Negotiate` method

The `Context.Negotiate` method used for serving different representations of a resource at the same URI. It returns `context.ErrContentNotSupported` when not matched mime types.

```go
Negotiate(v interface{}) (int, error)
```

The `Context.Negotiate` method accepts an `interface{}` which can be any Go value or a [ContentNegotiator](https://godoc.org/github.com/kataras/iris/context#ContentNegotiator) or a [ContentSelector](https://godoc.org/github.com/kataras/iris/context#ContentSelector) such as the [iris.N](https://godoc.org/github.com/kataras/iris/context#N) structure (see below).

* The "v" can be a single [iris.N](https://godoc.org/github.com/kataras/iris/context#N) struct value.
* The "v" can be any value completes the [context.ContentSelector](https://godoc.org/github.com/kataras/iris/context#ContentSelector) interface.
* The "v" can be any value completes the [context.ContentNegotiator](https://godoc.org/github.com/kataras/iris/context#ContentNegotiator) interface.
* The "v" can be any value of struct (JSON, JSONP, XML, YAML, MsgPack, Protobuf) or

  `string(TEXT, HTML)` or `[]byte(Markdown, Binary)` or `[]byte` with any matched mime type.

* If the "v" is nil, the `Context.Negotitation()` builder's content will be used instead, otherwise "v" overrides builder's content (server mime types are still retrieved by its registered, supported, mime list)
* Set mime type priorities by [Negotiation().MIME.Text.JSON.XML.HTML...]https://godoc.org/github.com/kataras/iris/context#NegotiationBuilder.JSON).
* Set charset priorities by [Negotiation().Charset(...)](https://godoc.org/github.com/kataras/iris/context#NegotiationBuilder.Charset).
* Set encoding algorithm priorities by [Negotiation().Encoding(...)](https://godoc.org/github.com/kataras/iris/context#NegotiationBuilder.Encoding).
* Modify the accepted by [Negotiation().Accept.Override().XML().JSON().Charset(...).Encoding(...)](https://godoc.org/github.com/kataras/iris/context#NegotiationAcceptBuilder.Override).

The `iris.N` is a struct which can be passed on the `Context.Negotiate` method.
It contains fields which should be filled based on the `Context.Negotiation()`
server side values. If no matched mime then its "Other" field will be sent,
which should be a string or []byte.
It completes the `ContentSelector` interface.

```go
type N struct {
	Text, HTML string
	Markdown   []byte
	Binary     []byte

	JSON     interface{}
	Problem  Problem
	JSONP    interface{}
	XML      interface{}
	YAML     interface{}
	Protobuf interface{}
	MsgPack  interface{}

	Other []byte // custom content types.
}
```

If the given `interface{}` value is not a type which implements one the above then the `Negotiate` method will render that based on the request's `Accept` header value matching the declared priorities. 

Note that if the given `v interface{}` is nil then it will uses the contents declared by the `Negotiation` builder itself.

### Usage

```go
func handler(ctx iris.Context) {
    // data := [...]
    ctx.Negotiation().
        JSON(data).
        XML(data).
        HTML("<h1>Test Name</h1><h2>Age 26</h2>").
        EncodingGzip().
        Charset("utf-8")

    err := ctx.Negotiate(nil)
    // [handle err]
}
```

## Example

When the client accepts **JSON** and **XML** and **HTML** responses from a specific server's endpoint and the server can render all of them:

```go
type testdata struct {
    ID   uint64 `json:"id" xml:"ID"`
    Name string `json:"name" xml:"Name"`
    Age  int    `json:"age" xml:"Age"`
}

func main() {
    users := app.Party("/users")
    users.Use(setAcceptTypes)

    users.Post("/{id:uint64}", handler)
    // [...]
}

func setAcceptTypes(ctx iris.Context) {
    ctx.Negotiation().JSON().XML().HTML().EncodingGzip()
    ctx.Next()
}

func handler(ctx iris.Context) {
    data := testdata{
        ID: ctx.Params().GetUint64Default("id", 0),
        Name: "Test Name",
        Age:  26,
    }

    ctx.Negotiate(iris.N{
        JSON: data,
        XML: data,
        HTML: "<h1>Test Name</h1><h2>Age 26</h2>",
    })
}
```

That's all, read the examples for a comprehensive understanding in practise.

<!-- slide:break-100 -->
