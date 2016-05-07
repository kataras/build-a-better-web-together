# TLS

```go
// Listen starts the standalone http server
// which listens to the addr parameter which as the form of
// host:port or just port
//
// It panics on error if you need a func to return an error use the ListenWithErr
// ex: iris.Listen(":8080")
Listen(addr string)

// ListenWithErr starts the standalone http server
// which listens to the addr parameter which as the form of
// host:port or just port
//
// It returns an error you are responsible how to handle this
// if you need a func to panic on error use the Listen
// ex: log.Fatal(iris.ListenWithErr(":8080"))
ListenWithErr(addr string) error

// ListenTLS Starts a https server with certificates,
// if you use this method the requests of the form of 'http://' will fail
// only https:// connections are allowed
// which listens to the addr parameter which as the form of
// host:port or just port
//
// It panics on error if you need a func to return an error use the ListenTLSWithErr
// ex: iris.ListenTLS(":8080","yourfile.cert","yourfile.key")
ListenTLS(addr string, certFile, keyFile string)

// ListenTLSWithErr Starts a https server with certificates,
// if you use this method the requests of the form of 'http://' will fail
// only https:// connections are allowed
// which listens to the addr parameter which as the form of
// host:port or just port
//
// It returns an error you are responsible how to handle this
// if you need a func to panic on error use the ListenTLS
// ex: log.Fatal(iris.ListenTLS(":8080","yourfile.cert","yourfile.key"))
ListenTLSWithErr(addr string, certFile, keyFile string) error

```
```go
iris.Listen(":8080")
log.Fatal(iris.ListenWithErr(":8080"))

iris.ListenTLS(":8080", "myCERTfile.cert", "myKEYfile.key")
log.Fatal(iris.ListenTLS(":8080", "myCERTfile.cert", "myKEYfile.key"))

```