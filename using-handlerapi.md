# Using Handler API
HandlerAPI is any custom struct which has an `*iris.Context` field.

Instead of writing Handlers/HandlerFuncs for eachone API routes, you can use the ` iris.API` function.

**For example**, for a user API you need some of these routes:

- GET `/users` , for selecting all
- GET`/users/:id` , for selecting specific
- PUT `/users` , for inserting
- POST `/users/:id` , for updating
- DELETE `/users/:id` , for deleting

Normally, with HandlerFuncs you should do something like this:

```go
iris.Get("/users", func(ctx *iris.Context){})
iris.Get("/users/:id", func(ctx *iris.Context){ id := ctx.Param("id) })

iris.Put("/users",...)

iris.Post("/users/:id", ...)

iris.Delete("/users/:id", ...)
```




