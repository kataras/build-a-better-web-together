# Custom HTTP Errors

You can define your own handlers for http errors, which can render an html file for example. e.g for for 404 not found:

```go
	iris.OnError(404,func (c *iris.Context){
		c.WriteHTML(iris.StatusOK, "<h1> The page you looking doesn't exists </h1>")
		c.SetStatusCode(404)
	})
	//or OnNotFound(func (c *iris.Context){})... for 404 only.
	//or OnPanic(func (c *iris.Context){})... for 500 only.
}

```

We saw how to declare a custom error for a http status code, now let's look for how to send/emit an error to the client manually  , for example let's emit the 404 we defined before, simple:

```go

iris.Get("/thenotfound",func (c *iris.Context) {
	c.EmitError(404)
	//or c.NotFound() for 404 only.
	//and c.Panic() for 500 only.
})

```