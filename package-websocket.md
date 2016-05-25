# Websockets


[This is a package](https://github.com/kataras/iris/tree/master/websocket)

**WebSocket is a protocol providing full-duplex communication channels over a single TCP connection**. The WebSocket protocol was standardized by the IETF as RFC 6455 in 2011, and the WebSocket API in Web IDL is being standardized by the W3C.

WebSocket is designed to be implemented in web browsers and web servers, but it can be used by any client or server application. The WebSocket Protocol is an independent TCP-based protocol. Its only relationship to HTTP is that its handshake is interpreted by HTTP servers as an Upgrade request. The WebSocket protocol makes more interaction between a browser and a website possible, **facilitating the real-time data transfer from and to the server**. 

[Read more about Websockets](https://en.wikipedia.org/wiki/WebSocket)

-----

How to use

**Server-side**
```go
import "github.com/kataras/iris/websocket"
//...

// important staff

w := websocket.New(api, "/my_endpoint")
// for default 'iris.' station use that: w := websocket.New(iris.DefaultIris, "/my_endpoint")

w.OnConnection(func(c websocket.Connection) {
	c.On("chat", func(message string) {
		c.To(websocket.Broadcast).Emit("chat", "Message from: "+c.ID()+"-> "+message) // to all except this connection
		// c.To("to a specific connection.ID() [rooms are coming soon]").Emit...
		c.Emit("chat", "Message from myself: "+message)
	})
})

//


// ...

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
View a working example by pressing [here](https://github.com/iris-contrib/examples/tree/master/websocket)