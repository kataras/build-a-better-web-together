# Custom responses

You can easily write custom response by using custom serialize engines \(see next chapter\) or by using iris.Context directly.

For instance, to return a custom content type and content via handler, you could define a handler function like:

```go
iris.Get("/testing", func(ctx *iris.Context) {
 ctx.SetContentType("text/plain")
 ctx.SetBody("something special in the body")
})
```

