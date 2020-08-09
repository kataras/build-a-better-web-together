# URL Query Parameters

The Iris Context has two methods that return the `net/http` standard `http.ResponseWriter` and `http.Request` values as we already mention at the previous chapters.

* `Context.Request()`
* `Context.ResponseWriter()`

However, except the unique Iris features and helpers that Iris Context offers, for easier development, we provide some wrappers of existing `net/http` capabilities as well.

This is the full list of methods that can help you when working with URL query string.

```go
// URLParam returns true if the url parameter exists, otherwise false.
URLParamExists(name string) bool
// URLParamDefault returns the get parameter from a request,
// if not found then "def" is returned.
URLParamDefault(name string, def string) string
// URLParam returns the get parameter from a request, if any.
URLParam(name string) string
// URLParamTrim returns the url query parameter with
// trailing white spaces removed from a request.
URLParamTrim(name string) string
// URLParamTrim returns the escaped url query parameter from a request.
URLParamEscape(name string) string
// URLParamInt returns the url query parameter as int value from a request,
// returns -1 and an error if parse failed.
URLParamInt(name string) (int, error)
// URLParamIntDefault returns the url query parameter as int value from a request,
// if not found or parse failed then "def" is returned.
URLParamIntDefault(name string, def int) int
// URLParamInt32Default returns the url query parameter as int32 value from a request,
// if not found or parse failed then "def" is returned.
URLParamInt32Default(name string, def int32) int32
// URLParamInt64 returns the url query parameter as int64 value from a request,
// returns -1 and an error if parse failed.
URLParamInt64(name string) (int64, error)
// URLParamInt64Default returns the url query parameter as int64 value from a request,
// if not found or parse failed then "def" is returned.
URLParamInt64Default(name string, def int64) int64
// URLParamFloat64 returns the url query parameter as float64 value from a request,
// returns -1 and an error if parse failed.
URLParamFloat64(name string) (float64, error)
// URLParamFloat64Default returns the url query parameter as float64 value from a request,
// if not found or parse failed then "def" is returned.
URLParamFloat64Default(name string, def float64) float64
// URLParamBool returns the url query parameter as boolean value from a request,
// returns an error if parse failed or not found.
URLParamBool(name string) (bool, error)
// URLParams returns a map of GET query parameters separated by comma if more than one
// it returns an empty map if nothing found.
URLParams() map[string]string
```

Query string parameters are parsed using the existing underlying request object. The request responds to a url matching: _/welcome?firstname=Jane&lastname=Doe_.

* `ctx.URLParam("lastname")` == `ctx.Request().URL.Query().Get("lastname")`

Example Code:

```go
    app.Get("/welcome", func(ctx iris.Context) {
        firstname := ctx.URLParamDefault("firstname", "Guest")
        lastname := ctx.URLParam("lastname") 

        ctx.Writef("Hello %s %s", firstname, lastname)
    })
```

<!-- slide:break-80 -->
