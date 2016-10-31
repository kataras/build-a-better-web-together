# Streaming


Do  progressive rendering via multiple flushes, streaming.


```go
// StreamWriter registers the given stream writer for populating the response body.
//
// This function may be used in the following cases:
//
//     * if response body is too big (more than 10MB).
//     * if response body is streamed from slow external sources.
//     * if response body must be streamed to the client in chunks.
//     (aka `http server push`).
StreamWriter(cb func(writer *bufio.Writer))
``` 


## Usage example
```go
package main

import(
	"gopkg.in/kataras/iris.v4"
	"bufio"
	"time"
	"fmt"
)

func main() {
	iris.Any("/stream",func (ctx *iris.Context){
		ctx.StreamWriter(stream)
	})

	iris.Listen(":8080")
}

func stream(w *bufio.Writer) {
	for i := 0; i < 10; i++ {
		fmt.Fprintf(w, "this is a message number %d", i)

		// Do not forget flushing streamed data to the client.
		if err := w.Flush(); err != nil {
			return
		}
		time.Sleep(time.Second)
	}
}

```

To achieve the oposite make use of the ` StreamReader`:
```go
// StreamReader sets the response body stream and optionally body size.
//
// If bodySize is >= 0, then the bodyStream must provide the exact bodySize bytes 
// before returning io.EOF.
//
// If bodySize < 0, then bodyStream is read until io.EOF.
//
// bodyStream.Close() is called after finishing reading all body data
// if it implements io.Closer.
//
// See also StreamReader.
StreamReader(bodyStream io.Reader, bodySize int)
```