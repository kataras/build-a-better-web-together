# Streaming


Fasthttp has very good support for doing progressive rendering via multiple flushes, streaming. Here is an example, taken from [here](https://github.com/valyala/fasthttp/blob/05949704db9b49a6fc7aa30220c983cc1c5f97a6/requestctx_setbodystreamwriter_example_test.go)

```go

package main

import(
	"github.com/kataras/iris"
	"bufio"
	"time"
	"fmt"
)

func main() {
	iris.Any("/stream",func (ctx *iris.Context){
		ctx.Stream(stream)
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