# Gzip

**Content-Type: _*_**

**Gzip** is a method of compressing files (making them smaller) for faster network transfers. It is also a file format. Compression allows your web server to provide smaller file sizes which load faster for your website users. Enabling **gzip compression** is a standard practice. https://en.wikipedia.org/wiki/Gzip

To enable **compression** on **all** server's **responses** and **requests** use the `iris.Compression` middleware:

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

> If the client does not accept an encoding then it will write the contents as they are, without compression.

When you want control over it you can use the `Context.CompressWriter(enable bool)` method before sent the response:

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
