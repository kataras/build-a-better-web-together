# Websockets

[WebSocket](https://wikipedia.org/wiki/WebSocket) is a protocol that enables two-way persistent communication channels over TCP connections. It is used for applications such as chat, stock tickers, games, anywhere you want real-time functionality in a web application.

Use WebSockets when you need to work directly with a socket connection. For example, you might need the best possible performance for a real-time game.

First of all, read the [**kataras/neffos wiki**](https://github.com/kataras/neffos/wiki) to master the new websocket library built for net/http and Iris.

It comes pre-installed with Iris, however you can install it separately by executing the following shell command.

```bash
$ go get github.com/kataras/neffos@latest
```

Continue by reading how you can register the neffos websocket server into your Iris application.

A list of comprehensive examples working with websockets can be found at: [https://github.com/kataras/iris/tree/master/\_examples/websocket](https://github.com/kataras/iris/tree/master/_examples/websocket).

The [iris/websocket](https://github.com/kataras/iris/tree/master/websocket) subpackage contains (only) Iris-specific migrations and helpers for the [neffos websocket framework](https://github.com/kataras/neffos) one.

For example, to gain access of the request's `Context` you can call the `websocket.GetContext(Conn)` from inside an event message handler/callback:

```go
// GetContext returns the Iris Context from a websocket connection.
func GetContext(c *neffos.Conn) Context
```

To register a websocket `neffos.Server` to a route use the `websocket.Handler` function:

```go
// IDGenerator is an iris-specific IDGenerator for new connections.
type IDGenerator func(Context) string

// Handler returns an Iris handler to be served in a route of an Iris application.
// Accepts the neffos websocket server as its first input argument
// and optionally an Iris-specific `IDGenerator` as its second one.
func Handler(s *neffos.Server, IDGenerator ...IDGenerator) Handler
```

**Usage**

```go
import (
    "github.com/kataras/neffos"
    "github.com/kataras/iris/v12/websocket"
)

// [...]

onChat := func(ns *neffos.NSConn, msg neffos.Message) error {
    ctx := websocket.GetContext(ns.Conn)
    // [...]
    return nil
}

app := iris.New()
ws := neffos.New(websocket.DefaultGorillaUpgrader, neffos.Namespaces{
    "default": neffos.Events {
        "chat": onChat,
    },
})
app.Get("/websocket_endpoint", websocket.Handler(ws))
```

<!-- slide:break-100 -->
