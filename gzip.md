# Gzip

Gzip compression is easy.


For **auto-gzip** to all response and template engines, 
just set the `iris.Config.Gzip = true`, which you can also change for specific render options:

```go
//...
context.Render("mytemplate.html", bindingStruct{}, iris.RenderOptions{"gzip": false})
context.Render("my-custom-response", iris.Map{"anything":"everything"} , iris.RenderOptions{"gzip": false}) 
```

```go
// WriteGzip writes response with gzipped body to w.
//
// The method gzips response body and sets 'Content-Encoding: gzip'
// header before writing response to w.
//
// WriteGzip doesn't flush response to w for performance reasons.
WriteGzip(w *bufio.Writer) error 


// WriteGzipLevel writes response with gzipped body to w.
//
// Level is the desired compression level:
//
//     * CompressNoCompression
//     * CompressBestSpeed
//     * CompressBestCompression
//     * CompressDefaultCompression
//
// The method gzips response body and sets 'Content-Encoding: gzip'
// header before writing response to w.
//
// WriteGzipLevel doesn't flush response to w for performance reasons.
WriteGzipLevel(w *bufio.Writer, level int) error
```

How to use
```go
iris.Get("/something", func(ctx *iris.Context){
    ctx.Response.WriteGzip(...) 
})

```

## Other

See [Static files](static-files.md) and learn how you can serve big files, assets or webpages with gzip compression.