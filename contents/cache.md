# Cache

Sometimes is important to cache routes that serve static content to make your web app performs faster and not spend time on re-building a response for clients.

There are two ways to achieve HTTP caching. One is to store the contents per handler on the server-side and the other is by checking some headers and send `304 not modified` in order to let the browser or any compatible client to handle the caching by itself.

Iris provides both server and client caching through its [iris/cache](https://github.com/kataras/iris/tree/master/cache) subpackage which exports the `iris.Cache` and `iris.Cache304` middlewares.

Let's see their outline.

## Cache

`Cache` is a middleware that provides server-side cache functionality to the next handlers, can be used as: `app.Get("/", iris.Cache, aboutHandler)`.

`Cache` accepts one single parameter: the cache expiration duration. If the expiration is invalid value, &lt;=2 seconds, then expiration is taken by the "cache-control's maxage" header

All types of response can be cached, templates, json, text, anything.

Use it for server-side caching, see the `Cache304` for an alternative approach that may be more suited to your needs.

```go
func Cache(expiration time.Duration) Handler
```

For more options and customization use the [kataras/iris/cache.Cache](https://godoc.org/github.com/kataras/iris/cache#Cache) which returns a [structure from which you can add or remove "Rules"](https://godoc.org/github.com/kataras/iris/cache/client#Handler) instead.

## NoCache

`NoCache` is a middleware which overrides the Cache-Control, Pragma and Expires headers in order to disable the cache during the browser's back and forward feature.

A good use of this middleware is on HTML routes; to refresh the page even on "back" and "forward" browser's arrow buttons.

```go
func NoCache(Context)
```

See `StaticCache` for the opposite behavior.

## StaticCache

`StaticCache` returns a middleware for caching static files by sending the "Cache-Control" and "Expires" headers to the client. It accepts a single input parameter, the "cacheDur", a time.Duration that it's used to calculate the expiration.

If "cacheDur" &lt;=0 then it returns the `NoCache` middleware instaed to disable the caching between browser's "back" and "forward" actions.

Usage: `app.Use(iris.StaticCache(24 * time.Hour))` or `app.Use(iris.StaticCache(-1))`.

```go
func StaticCache(cacheDur time.Duration)Handler
```

A middleware, which is a simple Handler, can be called inside another handler as well, example:

```go
 cacheMiddleware := iris.StaticCache(...)

 func(ctx iris.Context){
  cacheMiddleware(ctx)
  [...]
}
```

## Cache304

`Cache304` returns a middleware sends a `StatusNotModified` (304) whenever the "If-Modified-Since" request header (time) is before the time.Now() + expiresEvery (always compared to their UTC values).

Clients that are compatible with the HTTP RCF (all browsers are and tools like postman) will correctly handle the caching.

The only disadvantage of using that instead of server-side caching is that this method will send a 304 status code instead of 200, So, if you use it side by side with other micro services you have to check for that status code as well for a valid response.

Developers are free to extend this method's behavior by watching system directories changes manually and use of the `ctx.WriteWithExpiration` with a "modtime" based on the file modified date, similar to the `HandleDir`(which sends status OK\(200) and browser disk caching instead of 304\).

```go
func Cache304(expiresEvery time.Duration) Handler
```

Examples can be found at: [https://github.com/kataras/iris/tree/master/\_examples/response-writer/cache](https://github.com/kataras/iris/tree/master/_examples/response-writer/cache).

