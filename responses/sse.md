# SSE (Server-Sent Events - One Way Messaging)

**Content-Type: _*_**

A server-sent event is when a web page automatically gets updates from a server.

This was also possible before, but the web page would have to ask if any updates were available. With server-sent events, the updates come automatically.

Examples: Facebook/Twitter updates, stock price updates, news feeds, sport results, etc. https://www.w3schools.com/htmL/html5_serversentevents.asp

## Receive Server-Sent Event Notifications

The EventSource object is used to receive server-sent event notifications:

```js
var source = new EventSource("http://localhost:8080/handler");
source.onmessage = function(event) {
  document.getElementById("result").innerHTML += event.data + "<br>";
};
```

### Check Server-Sent Events Support

```js
if(typeof(EventSource) !== "undefined") {
    // Yes! Server-sent events support!
    // Some code.....
} else {
    // Sorry! No server-sent events support..
}
```

## Server-Side Code Example

The server-side event stream syntax is simple. Set the `"Content-Type"` header to `"text/event-stream"`. Now you can start sending event streams. The response messages starts with `"data: "` and ends with `"\n\n"`.

```go
func handler(ctx iris.Context) {
    // It's always recommended to first check for `Flusher`
    // compatibility in Go, you do that with:
	flusher, ok := ctx.ResponseWriter().Flusher()
	if !ok {
		ctx.StopWithText(iris.StatusHTTPVersionNotSupported, "Streaming unsupported!")
		return
    }

    ctx.ContentType("text/event-stream")
    ctx.Header("Cache-Control", "no-cache")

    now := time.Now()
    ctx.Writef("data: The server time is: %s\n\n", now)

    // Flush the data immediately instead of buffering it for later.
    flusher.Flush()
}
```

* Set the `"Content-Type"` header to `"text/event-stream"`
* Specify that the page should not cache
* Output the data to send (**Always** start with `"data: "`)
* `Flush` the output data back to the web page
