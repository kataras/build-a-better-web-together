# Compression

HTTP compression, otherwise known as content encoding, is a publicly defined way to compress textual content transferred from web servers to browsers. HTTP compression uses public domain compression algorithms, like **gzip** and **brotli**, to compress HTML, JavaScript, CSS, and other text files at the server. This standards-based method of delivering compressed content is built into HTTP 1.1, and most modern browsers that support HTTP 1.1 support ZLIB inflation of deflated documents. In other words, they can decompress compressed files automatically, which saves time and bandwidth. https://developer.mozilla.org/en-US/docs/Web/HTTP/Compression

In Iris you can enable the best offered compression with a single line of code:

```go
app.Use(iris.Compression)
```

The first, `iris.Compression` is a middleware which enables **writing** compressed data to the client and allows server to **read** compressed data from the client.

To enable **compression** on **all** server's **responses** and **requests**:

```go
func main() {
    app := iris.New()
    app.Use(iris.Compression)

    app.Post("/data", handler)
}

func handler(ctx iris.Context) {
    response := iris.Map{"name", "kataras"}
    ctx.JSON(response)
}
```

If you want to enable only writing and not reading compressed data, you can **skip** the `iris.Compression` and use the `ctx.CompressWriter(true)` alone, and visa-versa.

```go
func handler(ctx iris.Context) {
    ctx.CompressWriter(true)

   // [...]
}
```

There is also the `Context.ClientSupportsEncoding(s ...string) bool` method which reports if the client does accept and support gzip encoding:

```go
func handler(ctx iris.Context) {
    if ctx.ClientSupportsEncoding("gzip") {
        // client expects and supports gzip replies.
    } else {
        // client does not support gzip.
    }
}
```

That's all, your handlers will send and/or receive compressed data automatically, no further actions are required.

## Q/A

How Iris selects the compression algorithm to use on write?

* Iris reads the request's `Accept-Encoding` header value, if it contains more than one compression algorithm then it negotiates best candidate and compress data using the selected algorithm. By-default browsers accept the `gzip` one.

How Iris selects the algorithm to decompress the request body?

* Based on the request's `Content-Encoding` header value.

What are the builtin compression algorithms?

- ["gzip"](https://en.wikipedia.org/wiki/Gzip)
- ["deflate"](https://en.wikipedia.org/wiki/DEFLATE)
- "br" [brotli](https://en.wikipedia.org/wiki/Brotli)
- ["snappy"](https://en.wikipedia.org/wiki/Snappy_(compression)) and

How can I see the selected algorithm used to write and read?

* You have to `import "github.com/kataras/iris/v12/context"` package to access the Iris writer and reader types.

```go
func handler(ctx iris.Context) {
    if cw, ok := ctx.ResponseWriter().(*context.ResponseCompressWriter); ok {
        alg := cw.Encoding
        // [...]
    }

    if cr, ok := ctx.Request().Body.(*context.CompressReader); ok {
        alg := cr.Encoding
        // [...]
    }
}

```

<!-- slide:break-100 -->
