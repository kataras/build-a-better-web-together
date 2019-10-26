# Handle HTTP errors

You can define your own handlers when a specific http error code occurs.

> Error codes are the http status codes that are bigger than or equal to 400, like 404 not found and 500 internal server.

Example code:

```go
package main

import "github.com/kataras/iris/v12"

func main(){
    app := iris.New()
    app.OnErrorCode(iris.StatusNotFound, notFound)
    app.OnErrorCode(iris.StatusInternalServerError, internalServerError)
    // to register a handler for all "error"
    // status codes(kataras/iris/context.StatusCodeNotSuccessful)
    // defaults to < 200 || >= 400:
    // app.OnAnyErrorCode(handler)
    app.Get("/", index)
    app.Run(iris.Addr(":8080"))
}

func notFound(ctx iris.Context) {
    // when 404 then render the template
    // $views_dir/errors/404.html
    ctx.View("errors/404.html")
}

func internalServerError(ctx iris.Context) {
    ctx.WriteString("Oups something went wrong, try again")
}

func index(ctx iris.Context) {
    ctx.View("index.html")
}
```

> Learn more about [View](../view.md).

## The Problem type

Iris has builtin support for the [Problem Details for HTTP APIs](https://tools.ietf.org/html/rfc7807).

The `Context.Problem` writes a JSON or XML problem response. Behaves exactly like `Context.JSON` but with default ProblemOptions.JSON indent of " " and a response content type of "application/problem+json" instead.

Use the options.RenderXML and XML fields to change this behavior and
send a response of content type "application/problem+xml" instead.

```go
func newProductProblem(productName, detail string) iris.Problem {
    return iris.NewProblem().
        // The type URI, if relative it automatically convert to absolute.
        Type("/product-error"). 
        // The title, if empty then it gets it from the status code.
        Title("Product validation problem").
        // Any optional details.
        Detail(detail).
        // The status error code, required.
        Status(iris.StatusBadRequest).
        // Any custom key-value pair.
        Key("productName", productName)
        // Optional cause of the problem, chain of Problems.
        // .Cause(other iris.Problem)
}

func fireProblem(ctx iris.Context) {
    // Response like JSON but with indent of "  " and
    // content type of "application/problem+json"
    ctx.Problem(newProductProblem("product name", "problem details"),
        iris.ProblemOptions{
            // Optional JSON renderer settings.
            JSON: iris.JSON{
                Indent: "  ",
            },
            // OR
            // Render as XML:
            // RenderXML: true,
            // XML:       iris.XML{Indent: "  "},
            // Sets the "Retry-After" response header.
            //
            // Can accept:
            // time.Time for HTTP-Date,
            // time.Duration, int64, float64, int for seconds
            // or string for date or duration.
            // Examples:
            // time.Now().Add(5 * time.Minute),
            // 300 * time.Second,
            // "5m",
            //
            RetryAfter: 300,
            // A function that, if specified, can dynamically set
            // retry-after based on the request.
            // Useful for ProblemOptions reusability.
            // Overrides the RetryAfter field.
            //
            // RetryAfterFunc: func(iris.Context) interface{} { [...] }
    })
}
```

**Outputs "application/problem+json"**

```json
{
  "type": "https://host.domain/product-error",
  "status": 400,
  "title": "Product validation problem",
  "detail": "problem error details",
  "productName": "product name"
}
```

When `RenderXML` is set to `true` then the response will look be rendered as XML instead.

**Outputs "application/problem+xml"**

```xml
<Problem>
    <Type>https://host.domain/product-error</Type>
    <Status>400</Status>
    <Title>Product validation problem</Title>
    <Detail>problem error details</Detail>
    <ProductName>product name</ProductName>
</Problem>
```

Full example can be found at [_examples/routing/http-errors](https://github.com/kataras/iris/blob/master/_examples/routing/http-errors/main.go).
