# HTTP/2 Push

**Content-Type: _*_**

Server push lets the server preemptively "push" website assets
to the client without the user having explicitly asked for them.
When used with care, we can send what we know the user is going
to need for the page they’re requesting.

The **target** must either be an absolute path (like "/path") or an absolute
URL that contains a valid host and the same scheme as the parent request.
If the target is a path, it will inherit the scheme and host of the
parent request.

```go
Context.ResponseWriter().Push(target string, opts *http.PushOptions) error
```

> The `Push` method **returns** `iris.ErrPushNotSupported` if the client has disabled push or if push is not supported on the underlying connection.

## Example

The `Push` feature works only on **HTTP/2** servers.

Create the project structure, e.g.

```text
│   main.go
└───public
│      main.js
```

The `main.js` contains a simple alert function:

```js
window.alert("javascript loaded");
```

Execute the following command to generate sample server keys:

```sh
$ openssl req -new -newkey rsa:4096 -x509 -sha256 \
-days 365 -nodes -out mycert.crt -keyout mykey.key
```

Create the `main.go` file and copy-paste the code below:

```go
package main

import "github.com/kataras/iris/v12"

func main() {
	app := iris.New()
	app.Get("/", pushHandler)
	app.Get("/main.js", simpleAssetHandler)

	app.Run(iris.TLS("127.0.0.1:443", "mycert.crt", "mykey.key"))
}

func pushHandler(ctx iris.Context) {
	target := "/main.js"
	err := ctx.ResponseWriter().Push(target, nil)
	if err != nil {
		if err == iris.ErrPushNotSupported {
			ctx.StopWithText(iris.StatusHTTPVersionNotSupported,
				"HTTP/2 push not supported.")
		} else {
			ctx.StopWithError(iris.StatusInternalServerError, err)
		}
		return
	}

	ctx.HTML(`<html><body><script src="%s"></script></body></html>`, target)
}

func simpleAssetHandler(ctx iris.Context) {
	ctx.ServeFile("./public/main.js")
}
```

Run the server:

```sh
$ go run main.go
```

Now, open your browser's developer tools and click the `Network` tab. Navigate to https://127.0.0.1/,  the `main.js` _Initiator_ should be `Push / (index)` as shown below:

![](../.gitbook/assets/http2push.png)

<!-- slide:break-100 -->
