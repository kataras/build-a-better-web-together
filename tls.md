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
// ex: err := iris.ListenTo(":8080","yourfile.cert","yourfile.key")
ListenTLS(addr string, certFile string, keyFile string)

// ListenUNIX starts the process of listening to the new requests using a 'socket file', this works only on unix
//
// It panics on error if you need a func to return an error, use the ListenTo
// ex: err := iris.ListenTo(":8080", Mode: os.FileMode)
ListenUNIX(addr string, mode os.FileMode)

// ListenVirtual is useful only when you want to test Iris, it doesn't starts the server but it configures and returns it
// initializes the whole framework but server doesn't listens to a specific net.Listener
// it is not blocking the app
ListenVirtual(optionalAddr ...string) *Server

// Close terminates all the registered servers and returns an error if any
// if you want to panic on this error use the iris.Must(iris.Close())
Close() error 

```

```go
iris.Listen(":8080")
log.Fatal(iris.ListenWithErr(":8080"))

iris.ListenTLS(":8080", "myCERTfile.cert", "myKEYfile.key")
log.Fatal(iris.ListenTLSWithErr(":8080", "myCERTfile.cert", "myKEYfile.key"))

```

