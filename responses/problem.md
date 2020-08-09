# Problem

**Content-Type: "application/problem+json" or "application/problem+xml"**

Iris has builtin support for the [Problem Details for HTTP APIs](https://tools.ietf.org/html/rfc7807).

The `Context.Problem` method sends a `JSON` or `XML` problem response. Behaves exactly like `Context.JSON` but with default `ProblemOptions.JSON` indent of two spaces ("  ") and a response content type of `"application/problem+json"` instead.

```go
Problem(v interface{}, opts ...ProblemOptions) (int, error)
```

The `ProblemOptions` looks like that:

```go
// ProblemOptions the optional settings when server replies with a Problem.
// See `Context.Problem` method and `Problem` type for more details.
type ProblemOptions struct {
	// JSON are the optional JSON renderer options.
	JSON JSON

	// RenderXML set to true if want to render as XML doc.
	// See `XML` option field too.
	RenderXML bool
	// XML are the optional XML renderer options.
	// Affect only when `RenderXML` field is set to true.
	XML XML

	// RetryAfter sets the Retry-After response header.
	// https://tools.ietf.org/html/rfc7231#section-7.1.3
	// The value can be one of those:
	// time.Time
	// time.Duration for seconds
	// int64, int, float64 for seconds
	// string for duration string or for datetime string.
	//
	// Examples:
	// time.Now().Add(5 * time.Minute),
	// 300 * time.Second,
	// "5m",
	// 300
	RetryAfter interface{}
	// A function that, if specified, can dynamically set
	// retry-after based on the request. Useful for ProblemOptions reusability.
	// Should return time.Time, time.Duration, int64, int, float64 or string.
	//
	// Overrides the RetryAfter field.
	RetryAfterFunc func(Context) interface{}
}
```

Use the `options.RenderXML` and `XML` fields to change this behavior and send a response of content type `"application/problem+xml"` instead.

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
	response := newProductProblem("product name", "problem details")
	options := iris.ProblemOptions{
		JSON: iris.JSON{
			Indent: "  ",
		},
		RetryAfter: 300,
	}

	ctx.Problem(response, options)
}
```

**Outputs** "application/problem+json"

```json
{
  "type": "https://host.domain/product-error",
  "status": 400,
  "title": "Product validation problem",
  "detail": "problem error details",
  "productName": "product name"
}
```

When RenderXML is set to true then the response will look be rendered as XML instead.

**Outputs** "application/problem+xml"

```xml
<Problem>
    <Type>https://host.domain/product-error</Type>
    <Status>400</Status>
    <Title>Product validation problem</Title>
    <Detail>problem error details</Detail>
    <ProductName>product name</ProductName>
</Problem>
```

