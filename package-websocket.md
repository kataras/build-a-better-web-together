# Websockets


[This is a package](https://github.com/kataras/iris/tree/master/websocket)

**WebSocket is a protocol providing full-duplex communication channels over a single TCP connection**. The WebSocket protocol was standardized by the IETF as RFC 6455 in 2011, and the WebSocket API in Web IDL is being standardized by the W3C.

WebSocket is designed to be implemented in web browsers and web servers, but it can be used by any client or server application. The WebSocket Protocol is an independent TCP-based protocol. Its only relationship to HTTP is that its handshake is interpreted by HTTP servers as an Upgrade request. The WebSocket protocol makes more interaction between a browser and a website possible, **facilitating the real-time data transfer from and to the server**. 

[Read more about Websockets](https://en.wikipedia.org/wiki/WebSocket)

-----

## Configuration

```go
type Websocket struct {
	// WriteTimeout time allowed to write a message to the connection.
	// Default value is 10 * time.Second
	WriteTimeout time.Duration
	// PongTimeout allowed to read the next pong message from the connection
	// Default value is 60 * time.Second
	PongTimeout time.Duration
	// PingPeriod send ping messages to the connection with this period. Must be less than PongTimeout
	// Default value is (PongTimeout * 9) / 10
	PingPeriod time.Duration
	// MaxMessageSize max message size allowed from connection
	// Default value is 1024
	MaxMessageSize int
	// Endpoint is the path which the websocket server will listen for clients/connections
	// Default value is empty string, if you don't set it the Websocket server is disabled.
	Endpoint string
}

```

```go
iris.Config().Websocket

```

## Outline
websocket.Server / iris.Websocket() 
```go
OnConnection(func(c websocket.Connection){})
```

websocket.Connection
```go

// Receive from the client
On("anyCustomEvent", func(message string) {})
On("anyCustomEvent", func(message int){})
On("anyCustomEvent", func(message bool){})
On("anyCustomEvent", func(message anyCustomType){})
On("anyCustomEvent", func(){})

// Receive a native websocket message from the client
// compatible without need of import the iris-ws.js to the .html
OnMessage("anyMessage",func(message string){})

// Send to the client
Emit("anyCustomEvent", string)
Emit("anyCustomEvent", int)
Emit("anyCustomEvent", bool)
Emit("anyCustomEvent", anyCustomType)

// Send via native websocket way, compatible without need of import the iris-ws.js to the .html
EmitMessage("anyMessage")

// Send to specific client(s)
To("otherConnectionId").Emit/EmitMessage...
To("anyCustomRoom").Emit/EmitMessage...

// Send to all opened connections/clients
To(websocket.All).Emit/EmitMessage...

// Send to all opened connections/clients EXCEPT this client(c)
To(websocket.NotMe).Emit/EmitMessage...

// Rooms, group of connections/clients
Join("anyCustomRoom")
Leave("anyCustomRoom")


// Fired when the connection is closed
OnDisconnect(func(){})

```

## How to use

**Server-side**
```go
package main

import (
	"fmt"

	"github.com/kataras/iris"
	"github.com/kataras/iris/websocket"
)

type clientPage struct {
	Title string
	Host  string
}

func main() {

	iris.Static("/js", "./static/js", 1)

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Render("client.html", clientPage{"Client Page", ctx.HostString()})
	})

	// the path which the websocket client should listen/registed to ->
	iris.Config().Websocket.Endpoint = "/my_endpoint"

	ws := iris.Websocket() // get the websocket server

	var myChatRoom = "room1"
	ws.OnConnection(func(c websocket.Connection) {

		c.Join(myChatRoom)

		c.On("chat", func(message string) {
			// to all except this connection ->
			//c.To(websocket.Broadcast).Emit("chat", "Message from: "+c.ID()+"-> "+message)

			// to the client ->
			//c.Emit("chat", "Message from myself: "+message)

			//send the message to the whole room,
			//all connections are inside this room will receive this message
			c.To(myChatRoom).Emit("chat", "From: "+c.ID()+": "+message)
		})

		c.OnDisconnect(func() {
			fmt.Printf("\nConnection with ID: %s has been disconnected!", c.ID())
		})
	})

	fmt.Println("Server is listening at: 8080")
	iris.Listen(":8080")
}

```

**Client-side**

```js
// js/chat.js
var messageTxt;
var messages;

$(function () {

	messageTxt = $("#messageTxt");
	messages = $("#messages");


	ws = new Ws("ws://" + HOST + "/my_endpoint");
	ws.OnConnect(function () {
		console.log("Websocket connection enstablished");
	});

	ws.OnDisconnect(function () {
		appendMessage($("<div><center><h3>Disconnected</h3></center></div>"));
	});

	ws.On("chat", function (message) {
		appendMessage($("<div>" + message + "</div>"));
	})

	$("#sendBtn").click(function () {
		//ws.EmitMessage(messageTxt.val());
		ws.Emit("chat", messageTxt.val().toString());
		messageTxt.val("");
	})

})


function appendMessage(messageDiv) {
    var theDiv = messages[0]
    var doScroll = theDiv.scrollTop == theDiv.scrollHeight - theDiv.clientHeight;
    messageDiv.appendTo(messages)
    if (doScroll) {
        theDiv.scrollTop = theDiv.scrollHeight - theDiv.clientHeight;
    }
}

```


```html

<html>

<head>
	<title>My iris-ws</title>
</head>

<body>
	<div id="messages" style="border-width:1px;border-style:solid;height:400px;width:375px;">

	</div>
	<input type="text" id="messageTxt" />
	<button type="button" id="sendBtn">Send</button>
	<script type="text/javascript">
		var HOST = {{.Host}}
	</script>
	<script src="js/vendor/jquery-2.2.3.min.js" type="text/javascript"></script>
	<!-- /iris-ws.js is served automatically by the server -->
	<script src="/iris-ws.js" type="text/javascript"></script>
	<!-- -->
	<script src="js/chat.js" type="text/javascript"></script>
</body>

</html>


```


View a working example by navigating [here](https://github.com/iris-contrib/examples/tree/master/websocket) and if you need more than one websocket server [click here](https://github.com/iris-contrib/examples/tree/master/websocket_unlimited_servers)