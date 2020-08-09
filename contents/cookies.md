# Cookies

Cookies are accessible through the Request instance of Context. The `ctx.Request()` returns a `net/http#Request`.

However the Iris `Context` provides some helpers to make the most common use cases of cookies easier accessible to you and without any custom additional code of yours that would be required if you just using the Request's Cookies methods.

## Set a Cookie

The `SetCookie` and `UpsertCookie` methods add or set (replaces if necessary) a cookie.

The last argument of "options" is optional. a `CookieOption` can be used to modify the "cookie". You'll see later on what the available options are, custom ones can be added depending on your web application requirements, this also helps to not repeat yourself in a codebase.

Before we continue, the `Context` has a `SetSameSite(http.SameSite)` method too. It sets the ["SameSite"](https://web.dev/samesite-cookies-explained/) cookie field for all cookies to be sent.

```go
SetCookie(cookie *http.Cookie, options ...CookieOption)
```

If you want you can also use the `SetCookieKV` method which does not require an import of the `net/http` package.

```go
SetCookieKV(name, value string, options ...CookieOption)
```

Note that the default expiration for a cookie set by `SetCookieKV` is 365 days. You can either use the `CookieExpires` Cookie Option(see below) or globally by setting the `kataras/iris/Context.SetCookieKVExpiration` package-level variable.

The `CookieOption` is just a type for `func(*http.Cookie)`.

**Set Path**

```go
CookiePath(path string) CookieOption
```

**Set Expiration**

```go
iris.CookieExpires(durFromNow time.Duration) CookieOption
```

**HttpOnly**

```go
iris.CookieHTTPOnly(httpOnly bool) CookieOption
```

> HttpOnly field defaults to true for `RemoveCookie` and `SetCookieKV`.

And let's go a bit further with cookie encoding.

**Encode**

Provides encoding functionality when adding a cookie.

Accepts a `CookieEncoder` and sets the cookie's value to the encoded value.

Users of that is the `SetCookie` and `SetCookieKV`.

```go
iris.CookieEncode(encode CookieEncoder) CookieOption
```

**Decode**

Provides decoding functionality when retrieving a cookie.

Accepts a `CookieDecoder` and sets the cookie's value to the decoded value before return by the `GetCookie`.

User of that is the `GetCookie`.

```go
iris.CookieDecode(decode CookieDecoder) CookieOption
```

Where `CookieEncoder` can be described as:

A CookieEncoder should encode the cookie value.

* Should accept the cookie's name as its first argument and
* as second argument the cookie value ptr.
* Should return an encoded value or an empty one if encode operation failed.
* Should return an error if encode operation failed.

```go
CookieEncoder func(cookieName string, value interface{}) (string, error)
```

And `CookieDecoder`:

CookieDecoder should decode the cookie value.

* Should accept the cookie's name as its first argument,
* as second argument the encoded cookie value and as third argument the decoded value ptr.
* Should return a decoded value or an empty one if decode operation failed.
* Should return an error if decode operation failed.

```go
CookieDecoder func(cookieName string, cookieValue string, v interface{}) error
```

Errors are not printed, so you have to know what you're doing, and remember: if you use AES it only supports key sizes of 16, 24 or 32 bytes.

You either need to provide exactly that amount or you derive the key from what you type in.

## Get a Cookie

The `GetCookie` returns cookie's value by its name, returns empty string if nothing was found.

```go
GetCookie(name string, options ...CookieOption) string
```

If you want more than the value then use the following instead:

```go
cookie, err := ctx.Request().Cookie("name")
```

## Get all Cookies

The `ctx.Request().Cookies()` method returns a slice of all the available request cookies. Sometimes you want to modify them or perform an action on each one of them, the easiet way to do that is by the `VisitAllCookies` method.

```go
VisitAllCookies(visitor func(name string, value string))
```

## Remove a Cookie

The `RemoveCookie` method deletes a cookie by its name and path = "/", the root one.

Tip: change the cookie's path to the current one by providing the `iris.CookieCleanPath` Cookie Options, as : `RemoveCookie("nname", iris.CookieCleanPath)`.

Also, note that the default behavior is to set its `HttpOnly` to true. It performs a removal of cookie based on the web standards.

```go
RemoveCookie(name string, options ...CookieOption)
```

<!-- slide:break-80 -->
