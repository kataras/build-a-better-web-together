# MessagePack

**Content-Type: "application/msgpack"**

MessagePack is an efficient binary serialization format. It lets you exchange data among multiple languages like JSON. But it's faster and smaller. Small integers are encoded into a single byte, and typical short strings require only one extra byte in addition to the strings themselves. https://msgpack.org

The `Context.MsgPack(v)` is the method which sends MessagePack responses to the client. It accepts any Go value. As always, if it's a struct value, the fields should be exported.

```go
type Data struct {
	Message string `msgpack:"message"`
}

func handler(ctx iris.Context) {
    response := Data{Message: "a message"}
    ctx.MsgPack(response)
}
```

**Result**

```text
\x81\xa7message\xa9a message
```

<!-- slide:break-100 -->
