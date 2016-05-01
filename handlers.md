# Handlers

Handlers should implement the Handler interface:

```go
type Handler interface {
	Serve(*Context)
}
```