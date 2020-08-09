# Outputs

* if the return value is `string` then it will send that string as the response's body.
* If it's an `int` then it will send it as a status code.
* If it's an `error` then it will set a bad request with that error as its reason.
* If it's an `error` and an `int` then the error code is the output integer instead of 400(bad request).
* If it's a custom `struct` then it sent as a JSON, when a Content-Type header is not already set.
* If it's a custom `struct` and a `string` then the second output value, string, it will be the Content-Type and so on.

| Type | Replies to |
| :--- | :--- |
| string | body |
| string, string | content-type, body |
| string, int | body, status code |
| int | status code |
| int, string | status code, body |
| error | if not nil, bad request |
| any, bool | if false then fires not found |
| &lt;Τ&gt; | JSON body |
| &lt;Τ&gt;, string | body, content-type |
| &lt;Τ&gt;, error | JSON body or bad request |
| &lt;Τ&gt;, int | JSON body, status code |
| [Result](https://godoc.org/github.com/kataras/iris/hero#Result) | calls its `Dispatch` method |
| [PreflightResult](https://godoc.org/github.com/kataras/iris/hero#PreflightResult) | calls its `Preflight` method |

> Where `<T>` means any struct value.

