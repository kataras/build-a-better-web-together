# Outroduction

Iris does offer quite lot amount of helpers to help you out correctly send rich responses to the client. However, you are not limited to those helpers. You can send any data format to the client through `ctx.ContentType("a-mime-type")` and `ctx.Write([]byte)`.

Available Content Type helpers:

```go
// ContentType sets the response writer's
// header "Content-Type" to the 'cType'.
ContentType(cType string)
// GetContentType returns the response writer's
// header value of "Content-Type".
GetContentType() string
// GetContentType returns the request's
// trim-ed(without the charset and priority values)
// header value of "Content-Type".
GetContentTypeRequested() string

// GetContentLength returns the request's
// header value of "Content-Length".
GetContentLength() int64
```

Available raw `[]byte` Write helpers:

```go
Write(body []byte) (int, error)
// Writef formats according to a format specifier and writes to the response.
Writef(format string, args ...interface{}) (int, error)
// WriteString writes a simple string to the response.
WriteString(body string) (int, error)
// WriteNotModified sends a 304 "Not Modified" status code to the client,
// it makes sure that the content type, the content length headers
// and any "ETag" are removed before the response sent.
WriteNotModified()
// WriteWithExpiration works like `Write` but it will
// check if a resource is modified,
// based on the "modtime" input argument,
// otherwise sends a 304 status code in order to
// leave the client-side render the cached content.
WriteWithExpiration(body []byte, modtime time.Time) (int, error)
```
