# Websockets

[WebSocket](https://wikipedia.org/wiki/WebSocket) is a protocol that enables two-way persistent communication channels over TCP connections. It is used for applications such as chat, stock tickers, games, anywhere you want real-time functionality in a web application.

Use WebSockets when you need to work directly with a socket connection. For example, you might need the best possible performance for a real-time game.

First of all, read the **[kataras/neffos wiki](https://github.com/kataras/neffos/wiki)** to master the new websocket library built for net/http and Iris.

It comes pre-installed with Iris, however you can install it separately by executing the following shell command.

```sh
$ go get github.com/kataras/neffos@v0.0.8
```

Continue by reading how you can register the neffos websocket server into your Iris application.

A list of comprehensive examples working with websockets can be found at: <https://github.com/kataras/iris/tree/master/_examples/websocket>.

-----------

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

## Websocket Controller

Iris has a trivial way of registering websocket events via a Go structure. The websocket controller is part of the [MVC](mvc.md) features.

Iris has its own `iris/mvc/Application.HandleWebsocket(v interface{}) *neffos.Struct` to register controllers in existing Iris MVC applications(offering a fully featured dependency injection container for request values and static services).

```go
// HandleWebsocket handles a websocket specific controller.
// Its exported methods are the events.
// If a "Namespace" field or method exists then namespace is set,
// otherwise empty namespace will be used for this controller.
//
// Note that a websocket controller is registered and ran under
// a connection connected to a namespace
// and it cannot send HTTP responses on that state.
// However all static and dynamic dependencies behave as expected.
func (*mvc.Application) HandleWebsocket(controller interface{}) *neffos.Struct
```

Let's see a usage example, we want to bind the `OnNamespaceConnected`, `OnNamespaceDisconnect` built-in events and a custom `"OnChat"` event with our controller's methods.

**1.** We create the controller by declaring a NSConn type field as `stateless` and write the methods we need.

```go
type websocketController struct {
    *neffos.NSConn `stateless:"true"`
    Namespace string

    Logger MyLoggerInterface
}

func (c *websocketController) OnNamespaceConnected(msg neffos.Message) error {
	return nil
}

func (c *websocketController) OnNamespaceDisconnect(msg neffos.Message) error {
	return nil
}

func (c *websocketController) OnChat(msg neffos.Message) error {
    return nil
}
```

Iris is smart enough to catch the `Namespace string` struct field to use it to register the controller's methods as events for that namespace, alternatively you can create a controller method of `Namespace() string { return "default" }` or use the `HandleWebsocket`'s return value to `.SetNamespace("default")`, it's up to you.

**2.** We inititalize our MVC application targets to a websocket endpoint, as we used to do with regular HTTP Controllers for HTTP routes.

```go
import (
    // [...]
    "github.com/kataras/iris/v12/mvc"
)
// [app := iris.New...]

mvcApp := mvc.New(app.Party("/websocket_endpoint"))
```

**3.** We register our dependencies, if any.

```go
mvcApp.Register(
    &prefixedLogger{prefix: "DEV"},
)
```

**4.** We register one or more websocket controllers, each websocket controller maps to one namespace (just one is enough, as in most of the cases you don't need more, but that depends on your app's needs and requirements).

```go
mvcApp.HandleWebsocket(&websocketController{Namespace: "default"})
```

**5.** Next, we continue by mapping the mvc application as a connection handler to a websocket server (you may use more than one mvc applications per websocket server via `neffos.JoinConnHandlers(mvcApp1, mvcApp2)`).

```go
websocketServer := neffos.New(websocket.DefaultGorillaUpgrader, mvcApp)
```

**6.** And the last step is to register that server to our endpoint through a normal `.Get` method.

```go
mvcApp.Router.Get("/", websocket.Handler(websocketServer))
```
