# TLS

```go
// Listen starts the standalone http server
// which listens to the addr parameter which as the form of
// host:port
//
// It panics on error if you need a func to return an error, use the ListenTo
// ex: err := iris.ListenTo(config.Server{ListeningAddr:":8080"})
Listen(addr string) 

// ListenTLS Starts a https server with certificates,
// if you use this method the requests of the form of 'http://' will fail
// only https:// connections are allowed
// which listens to the addr parameter which as the form of
// host:port
//
// It panics on error if you need a func to return an error, use the ListenTo
// ex: err := iris.ListenTo(config.Server{":8080","yourfile.cert","yourfile.key"})
ListenTLS(addr string, certFile string, keyFile string)

// ListenTLSAuto starts a server listening at the specific nat address
// using key & certification taken from the letsencrypt.org 's servers
// it also starts a second 'http' server to redirect all 'http://$ADDR_HOSTNAME:80' to the' https://$ADDR'
//
// Notes:
// if you don't want the last feature you should use this method:
// iris.ListenTo(config.Server{ListeningAddr: "mydomain.com:443", AutoTLS: true})
// it's a blocking function
// Limit : https://github.com/iris-contrib/letsencrypt/blob/master/lets.go#L142
//
// example: https://github.com/iris-contrib/examples/blob/master/letsencyrpt/main.go
ListenTLSAuto(addr string)

// ListenUNIX starts the process of listening to the new requests using a 'socket file', this works only on unix
//
// It panics on error if you need a func to return an error, use the ListenTo
// ex: err := iris.ListenTo(config.Server{":8080", Mode: os.FileMode})
ListenUNIX(addr string, mode os.FileMode)

// ListenVirtual is useful only when you want to test Iris, it doesn't starts the server but it configures and returns it
// initializes the whole framework but server doesn't listens to a specific net.Listener
// it is not blocking the app
ListenVirtual(optionalAddr ...string) *Server

// ListenTo listens to a server but accepts the full server's configuration
// returns an error, you're responsible to handle that
// or use the iris.Must(iris.ListenTo(config.Server{}))
//
// it's a blocking func
ListenTo(cfg config.Server) (err error) 

// Close terminates all the registered servers and returns an error if any
// if you want to panic on this error use the iris.Must(iris.Close())
Close() error 

```

```go
iris.Listen(":8080")
err := iris.ListenTo(config.Server{ListeningAddr: ":8080"})
```
```go
iris.ListenTLS(":8080", "myCERTfile.cert", "myKEYfile.key")
err := iris.ListenTo(config.Server{ListeningAddr: ":8080", CertFile: "myCERTfile.cert", KeyFile: "myKEYfile.key"})
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
	// it also starts a second 'http://' server which will redirect all 'http://$PATH' requests to 'https://$PATH'
	iris.ListenTLSAuto("127.0.0.1:443")
}


```

