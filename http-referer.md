# HTTP Referer

The HTTP referer(originally a misspelling of **referrer**) is an optional HTTP header field that identifies the address of the webpage (i.e. the URI or IRI) which is linked to the resource being requested. By checking the referrer, the new webpage can see where the request originated.

Read more at [wikipedia](https://en.wikipedia.org/wiki/HTTP_referer)

Iris uses the [Shopify's goreferrer](https://github.com/Shopify/goreferrer/pull/27) package to expose the `Context.GetReferrer()` method.

The `GetReferrer` Context's method extracts and returns the information from the `"Referer"` header as specified in [https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy) or by the URL `query parameter "referer"`.

```go
GetReferrer() Referrer
```

Which `Referrer` looks like that:

```go
type (
    Referrer struct {
        Type       ReferrerType
        Label      string
        URL        string
        Subdomain  string
        Domain     string
        Tld        string         
        Path       string              
        Query      string                 
        GoogleType ReferrerGoogleSearchType
    }
```

The `ReferrerType` is the enum for a Referrer.Type value (indirect, direct, email, search, social). The available types are:

```go
ReferrerInvalid
ReferrerIndirect
ReferrerDirect
ReferrerEmail
ReferrerSearch
ReferrerSocial
```

The `GoogleType` can be one of those:

```go
ReferrerNotGoogleSearch
ReferrerGoogleOrganicSearch
ReferrerGoogleAdwords
```

## Example

```go
package main

import "github.com/kataras/iris/v12"

func main() {
    app := iris.New()

    app.Get("/", func(ctx iris.Context) {
        r := ctx.GetReferrer()
        switch r.Type {
        case iris.ReferrerSearch:
            ctx.Writef("Search %s: %s\n", r.Label, r.Query)
            ctx.Writef("Google: %s\n", r.GoogleType)
        case iris.ReferrerSocial:
            ctx.Writef("Social %s\n", r.Label)
        case iris.ReferrerIndirect:
            ctx.Writef("Indirect: %s\n", r.URL)
        }
    })

    app.Run(iris.Addr(":8080"))
}
```

How to `curl`:

```bash
curl http://localhost:8080?\
referer=https://twitter.com/Xinterio/status/1023566830974251008

curl http://localhost:8080?\
referer=https://www.google.com/search?q=Top+6+golang+web+frameworks\
&oq=Top+6+golang+web+frameworks
```

