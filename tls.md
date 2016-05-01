# TLS

```go
ListenTLS(fulladdr string, certFile, keyFile string) error
```
```go
log.Fatal(iris.ListenTLS(":8080", "myCERTfile.cert", "myKEYfile.key"))

```