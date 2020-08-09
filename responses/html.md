# HTML

**Content-Type: "text/html"**

The `Context.HTML(format, ...args)` is the method which sends HTML responses to the client. It accepts the value (like fmt package works).

```go
func handler(ctx iris.Context) {
    response := "Hello, <strong>%s!</strong>"
    ctx.HTML(response, "World")
}
```

**Result**

```html
Hello, <strong>World!</strong>
```

> If you want to learn how to render template files or embedded views please head over to the [View](../view) chapter instead.

<!-- slide:break-100 -->
