# Stream

**Content-Type: _*_**

**Chunked transfer encoding is a streaming data transfer mechanism** available in HTTP/1.1. In chunked transfer encoding, the data stream is divided into a series of non-overlapping "chunks". The chunks are sent out and received independently of one another. No knowledge of the data stream outside the currently-being-processed chunk is necessary for both the sender and the receiver at any given time.

The **chunked** keyword in the **Transfer-Encoding header** is used to indicate chunked transfer.

> HTTP/2 uses DATA frames to carry message payloads. The "chunked" transfer encoding defined in MUST NOT be used in HTTP/2.

References:
- [Wikipedia](https://en.wikipedia.org/wiki/Chunked_transfer_encoding)
- [RFC](https://tools.ietf.org/html/rfc7230#section-4.1)

As we read above, the `Transfer-Encoding: "chunked"` header is required:

```go
ctx.Header("Transfer-Encoding", "chunked")
```

Then, define the response content type (when we use the `ctx.Write` methods), e.g.

```go
ctx.ContentType("text/html")
```

And we write the contents:

```go
ctx.Writef("A message here")
```

Finally, we must flush the contents with:

```go
ctx.ResponseWriter().Flush()
```

And repeat.

Iris gives you a helper method which sends streaming data until connection close or error returned:

```go
StreamWriter(func(w io.Writer) error) error
```

The second way for streaming data is with `ctx.XXX` response methods (as we've seen previously, e.g. `ctx.JSON`) following by a `ctx.ResponseWriter().Flush()`).

## Example

Let's say that you want to send number messages from 1 to 29.

```go
var errDone = errors.New("done")

func handler(ctx iris.Context) {
    ctx.ContentType("text/html")
    ctx.Header("Transfer-Encoding", "chunked")
    i := 0
    ints := []int{1, 2, 3, 5, 7, 9, 11, 13, 15, 17, 23, 29}
    // Send the response in chunks and
    // wait for half a second between each chunk,
    // until connection closed.
    err := ctx.StreamWriter(func(w io.Writer) error {
        ctx.Writef("Message number %d<br>", ints[i])
        time.Sleep(500 * time.Millisecond) // simulate delay.
        if i == len(ints)-1 {
            return errDone // ends the loop.
        }
        i++
        return nil // continue write
    })

    if err != errDone {
        // Test it by canceling the request before the stream ends:
        // [ERRO] $DATETIME stream: context canceled.
        ctx.Application().Logger().Errorf("stream: %v", err)
    }
}
```

### JSON Example

Let's do the same using JSON.

```go
type messageNumber struct {
    Number int `json:"number"`
}

func handler(ctx iris.Context) {
    ctx.Header("Transfer-Encoding", "chunked")
    i := 0
    ints := []int{1, 2, 3, 5, 7, 9, 11, 13, 15, 17, 23, 29}
    // Send the response in chunks and wait
    // for half a second between each chunk,
    // until connection close.
    notifyClose := ctx.Request().Context().Done()
    for {
        select {
        case <-notifyClose:
            // err := ctx.Request().Context().Err()
            ctx.Application().Logger().Infof("Connection closed, loop end.")
            return
        default:
            ctx.JSON(messageNumber{Number: ints[i]})
            ctx.WriteString("\n")
            time.Sleep(500 * time.Millisecond) // simulate delay.
            if i == len(ints)-1 {
                ctx.Application().Logger().Infof("Loop end.")
                return
            }
            i++
            ctx.ResponseWriter().Flush()
        }
    }
}
```

**Result**

![](../.gitbook/assets/iris-stream-writer.gif)

That's all. As you've noticed, the client receives messages while loading. Check the next chapter (Server-Sent Events) to see an alternative way of sending messages to the client with connection-alive and loaded page.
