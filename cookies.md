## Cookies
Cookie management, even your little brother can do this!

```go
// SetCookie adds a cookie
SetCookie(cookie *fasthttp.Cookie)

// SetCookieKV adds a cookie, receives just a key(string) and a value(string)
SetCookieKV(key, value string)

// GetCookie returns cookie's value by it's name
// returns empty string if nothing was found
GetCookie(name string) string 

// RemoveCookie removes a cookie by it's name/key
RemoveCookie(name string) 
```
How to use 
```go

iris.Get("/set", func(c *iris.Context){
    c.SetCookieKV("name","iris")
    c.Write("Cookie has been setted.")
})

iris.Get("/get", func(c *iris.Context){
    name := c.GetCookie("name")
    c.Write("Cookie's value: %s", name)
})

iris.Get("/remove", func(c *iris.Context){
    if name := c.GetCookie("name"); name != "" {
       c.RemoveCookie("name")  
    }
    c.Write("Cookie has been removed.")
})

```