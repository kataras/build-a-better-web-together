# Text

**Content-Type: "text/plain"**

The `Context.Text(format, ...args)` is the method which sends plain text responses to the client. It accepts the value (like fmt package works). It's identical to `Context.WriteString` and `Context.Writef`.

```go
func handler(ctx iris.Context) {
    response := "Hello, %s!"
    ctx.Text(response, "World")
    // ctx.Writef(response, "World")
}
```

**Result**

```text
Hello, World!
```

That's all.
