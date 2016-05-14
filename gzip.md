# Gzip

Gzip compression is easy.


For **auto-gzip** to all rest and template responses, look the Gzip option at the iris.Config().Render.Rest.Gzip and iris.Config().Render.Template.Gzip [here](render.md)

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