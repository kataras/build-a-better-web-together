# JSONP

**Content-Type: "text/javascript"**

## What is JSONP, and why was it created?

Say you're on domain `example.com`, and you want to make a request to domain `example.net`. To do so, you need to cross domain boundaries, a no-no in most of browserland.

The one item that bypasses this limitation is `<script>` tags. When you use a script tag, the domain limitation is ignored, but under normal circumstances, you can't really do anything with the results, the script just gets evaluated.

Enter **JSONP**. When you make your request to a server that is JSONP enabled, you pass a special parameter that tells the server a little bit about your page. That way, the server is able to nicely wrap up its response in a way that your page can handle.

For example, say the server expects a parameter called callback to enable its JSONP capabilities. Then your request would look like:

```
http://www.example.net/sample?callback=mycallback
```

Without JSONP, this might return some basic JavaScript object, like so:
```json
{ "foo": "bar" }
```

However, with `JSONP`, when the server receives the "callback" parameter, it wraps up the result a little differently, returning something like this:

```js
mycallback({ "foo": "bar" });
```

As you can see, it will now invoke the method you specified. So, in your page, you define the callback function:

```js
mycallback = function(data){
  alert(data.foo);
};
```

And now, when the script is loaded, it'll be evaluated, and your function will be executed. Voila, cross-domain requests!

It's also worth noting the one major issue with JSONP: you lose a lot of control of the request. For example, there is no "nice" way to get proper failure codes back. As a result, you end up using timers to monitor the request, etc, which is always a bit suspect. The proposition for JSONRequest is a great solution to allowing cross domain scripting, maintaining security, and allowing proper control of the request.

**However**, CORS[*](https://github.com/kataras/iris/tree/master/_examples/auth/cors) is the recommended approach vs. JSONRequest. JSONP is still useful for older browser support, but given the security implications, unless you have no choice CORS is the better choice.

> https://stackoverflow.com/a/2067584 _(source)_

## Send JSONP with Iris

The `Context.JSONP(v, ...opts)` is the method which sends JSONP responses to the client. It accepts the value and optional settings for rendering. The `JSONP` options structure looks like this:

```go
type JSONP struct {
	Indent   string
	Callback string
}
```

> If `Indent` field is empty and the application runs without optimizations, the `Indent` field will be automatically set to `4 spaces`.

So, if we want to write a JSONP with indentation of two spaces and a callback extracted from URL Query Parameter of `?callback=mycallback`, we write something like that:

```go
func handler(ctx iris.Context) {
    callback := ctx.URLParamDefault("callback", "defaultCallback")
    response := map[string]interface{}{"foo": "bar"}
    options := iris.JSONP{Indent: "  ", Callback:true}
    ctx.JSONP(response, options)
}
```

If we want to render a Go struct as JSONP's callback data, the struct's fields we want to render should be **[exported](https://tour.golang.org/basics/3)**, and optionally tagged with the `json` struct tag. Look the exaple below:

```go
type Item struct {
    Name    string `json:"name"`
}

func handler(ctx iris.Context) {
    response := Item{
        Name: "gopher",
    }

    ctx.JSONP(response, iris.JSONP{Callback: "addToCard"})
}
```
