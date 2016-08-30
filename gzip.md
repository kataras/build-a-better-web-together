# Gzip

Gzip compression is easy.


Activate **auto-gzip** for all responses and template engines, 
just set `iris.Config.Gzip = true`. You can also enable gzipping for specific `Render()` calls:

```go
//...
context.Render("mytemplate.html", bindingStruct{}, iris.RenderOptions{"gzip": false})
context.Render("my-custom-response", iris.Map{"anything":"everything"} , iris.RenderOptions{"gzip": false}) 
```

```go
// WriteGzip writes the gzipped body of the response to w.
//
// Gzips the response body and sets the 'Content-Encoding: gzip'
// header before writing response to w.
//
// WriteGzip doesn't flush the response to w for performance reasons.
WriteGzip(w *bufio.Writer) error 


// WriteGzip writes the gzipped body of the response to w.
//
// Level is the desired compression level:
//
//     * CompressNoCompression
//     * CompressBestSpeed
//     * CompressBestCompression
//     * CompressDefaultCompression
//
// Gzips the response body and sets the 'Content-Encoding: gzip'
// header before writing response to w.
//
// WriteGzipLevel doesn't flush the response to w for performance reasons.
WriteGzipLevel(w *bufio.Writer, level int) error
```

How to use:
```go
iris.Get("/something", func(ctx *iris.Context){
    ctx.Response.WriteGzip(...) 
})

```

## Other

See [Static files](static-files.md) and learn how you can serve big files, assets or webpages with gzip compression.