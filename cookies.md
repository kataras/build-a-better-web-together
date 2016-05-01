## Cookies
Cookie management, even your little brother can do this!

```go
// SetCookie adds a cookie
SetCookie(cookie *fasthttp.Cookie)

// SetCookieKV adds a cookie, receives just a key(string) and a value(string)
SetCookieKV(key, value string)

// GetCookie returns cookie's value by it's name
GetCookie(name string) string 

// RemoveCookie removes a cookie by it's name/key
RemoveCookie(name string) 
```

How to use 
```go



```