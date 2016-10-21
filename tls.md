# Listen & Serve functions

```go
// Serve serves incoming connections from the given listener.
//
// Serve blocks until the given listener returns permanent error.
Serve(ln net.Listener) error

// Listen starts the standalone http server
// which listens to the addr parameter which as the form of
// host:port
//
// It panics on error if you need a func to return an error, use the Serve
Listen(addr string)

// ListenTLS Starts a https server with certificates,
// if you use this method the requests of the form of 'http://' will fail
// only https:// connections are allowed
// which listens to the addr parameter which as the form of
// host:port
//
// It panics on error if you need a func to return an error, use the Serve
// ex: iris.ListenTLS(":8080","yourfile.cert","yourfile.key")
ListenTLS(addr string, certFile, keyFile string)

// ListenLETSENCRYPT starts a server listening at the specific nat address
// using key & certification taken from the letsencrypt.org 's servers
// it's also starts a second 'http' server to redirect all 'http://$ADDR_HOSTNAME:80' to the' https://$ADDR'
// example: https://github.com/iris-contrib/examples/blob/master/letsencrypt/main.go
ListenLETSENCRYPT(addr string)

// ListenUNIX starts the process of listening to the new requests using a 'socket file', this works only on unix
//
// It panics on error if you need a func to return an error, use the Serve
// ex: iris.ListenUNIX(":8080", Mode: os.FileMode)
ListenUNIX(addr string, mode os.FileMode)

// Close terminates all the registered servers and returns an error if any
// if you want to panic on this error use the iris.Must(iris.Close())
Close() error


// Reserve re-starts the server using the last .Serve's listener
Reserve() error

// IsRunning returns true if server is running
IsRunning() bool
```

```go
// Serve
ln, err := net.Listen("tcp4", ":8080")
if err := iris.Serve(ln); err != nil {
   panic(err)
}
// same as api := iris.New(); api.Serve(ln), iris. contains a default iris instance, this exists for
// any function or field you will see at the rest of the gitbook.

// Listen
iris.Listen(":8080")

// ListenTLS
iris.ListenTLS(":8080", "./ssl/mycert.cert", "./ssl/mykey.key")

// and so on...
```

```go
// Package main provide one-line integration with letsencrypt.org
package main

import "github.com/kataras/iris"

func main() {
    iris.Get("/", func(ctx *iris.Context) {
        ctx.Write("Hello from SECURE SERVER!")
    })

    iris.Get("/test2", func(ctx *iris.Context) {
        ctx.Write("Welcome to secure server from /test2!")
    })

    // This will provide you automatic certification & key from letsencrypt.org's servers
    // it also starts a second 'http://' server which will redirect all 'http://$PATH'
    // requests to 'https://$PATH'
    iris.ListenLETSENCRYPT("127.0.0.1:443")
}
```

Examples:

* [Listen using a custom fasthttp server](https://github.com/iris-contrib/examples/tree/master/custom_fasthttp_server)
* [Serve content using a custom router](https://github.com/iris-contrib/examples/tree/master/custom_fasthttp_router)
* [Listen using a custom net.Listener](https://github.com/iris-contrib/examples/tree/master/custom_net_listener)
* [Redirect all http://$HOST to https://$HOST using a custom net.Listener](https://github.com/iris-contrib/examples/tree/master/listentls)
* [Listen using automatic ssl](https://github.com/iris-contrib/examples/tree/master/letsencrypt)
