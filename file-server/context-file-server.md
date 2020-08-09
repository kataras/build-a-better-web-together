# Serve Files from Context

```go
// SendFile sends a file as an attachment, that is downloaded and
// saved locally from client.
// Note that compression can be registered through `ctx.CompressWriter(true)`
// or `app.Use(iris.Compression)`.
// Use `ServeFile` if a file should be served as a page asset instead.
SendFile(filename string, destinationName string) error
// SendFileWithRate same as `SendFile` but it can throttle the speed of reading
// and though writing the file to the client.
SendFileWithRate(src, destName string, limit float64, burst int) error
```

**Usage**

Force-Send a file to the client:

```go
func handler(ctx iris.Context) {
	src := "./files/first.zip"
	ctx.SendFile(src, "client.zip")
}
```

Limit download speed to ~50Kb/s with a burst of 100KB:

```go
func handler(ctx iris.Context) {
    src := "./files/big.zip"
    // optionally, keep it empty to resolve the filename based on the "src".
	dest := "" 

	limit := 50.0 * iris.KB
	burst := 100 * iris.KB
	ctx.SendFileWithRate(src, dest, limit, burst)
}
```

```go
// ServeContent replies to the request using the content in the
// provided ReadSeeker. The main benefit of ServeContent over io.Copy
// is that it handles Range requests properly, sets the MIME type, and
// handles If-Match, If-Unmodified-Since, If-None-Match, If-Modified-Since,
// and If-Range requests.
//
// If the response's Content-Type header is not set, ServeContent
// first tries to deduce the type from name's file extension.
//
// The name is otherwise unused; in particular it can be empty and is
// never sent in the response.
//
// If modtime is not the zero time or Unix epoch, ServeContent
// includes it in a Last-Modified header in the response. If the
// request includes an If-Modified-Since header, ServeContent uses
// modtime to decide whether the content needs to be sent at all.
//
// The content's Seek method must work: ServeContent uses
// a seek to the end of the content to determine its size.
//
// If the caller has set w's ETag header formatted per RFC 7232, section 2.3,
// ServeContent uses it to handle requests using If-Match, If-None-Match, or If-Range.
//
// Note that *os.File implements the io.ReadSeeker interface.
// Note that compression can be registered through
// `ctx.CompressWriter(true)` or `app.Use(iris.Compression)`.
ServeContent(content io.ReadSeeker, filename string, modtime time.Time)
// ServeContentWithRate same as `ServeContent` but it can throttle the speed of reading
// and though writing the "content" to the client.
ServeContentWithRate(content io.ReadSeeker, filename string, modtime time.Time, limit float64, burst int)
// ServeFile replies to the request with the contents of the named
// file or directory.
//
// If the provided file or directory name is a relative path, it is
// interpreted relative to the current directory and may ascend to
// parent directories. If the provided name is constructed from user
// input, it should be sanitized before calling `ServeFile`.
//
// Use it when you want to serve assets like css and javascript files.
// If client should confirm and save the file use the `SendFile` instead.
// Note that compression can be registered through
// `ctx.CompressWriter(true)`
// or `app.Use(iris.Compression)`.
ServeFile(filename string) error
// ServeFileWithRate same as `ServeFile` but it can throttle the speed of reading
// and though writing the file to the client.
ServeFileWithRate(filename string, limit float64, burst int) error
```

**Usage**

```go
func handler(ctx iris.Context) {
	ctx.ServeFile("./public/main.js")
}
```

<!-- slide:break-100 -->
