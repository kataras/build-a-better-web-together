# Flash Messages

Sometimes you need to temporarily store data between requests of the same user, such as error or success messages after a form submission. The Iris sessions package supports flash messages.

As we've seen the [Sessions](https://github.com/kataras/iris-book/tree/949c38a02420899aa31dbb029b89f63a3b3846af/sessions/sessions/README.md) chapter, you initialize a session like this:

```go
import "github.com/kataras/iris/v12/sessions"

sess := sessions.New(sessions.Config{Cookie: "cookieName", ...})
```

```go
// [app := iris.New...]

app.Get("/path", func(ctx iris.Context) {
    session := sess.Start(ctx)
    // [...]
})
```

The `Session` contains the following methods to store, retrieve and remove flash messages.

```go
SetFlash(key string, value interface{})
HasFlash() bool
GetFlashes() map[string]interface{}

PeekFlash(key string) interface{}
GetFlash(key string) interface{}
GetFlashString(key string) string
GetFlashStringDefault(key string, defaultValue string) string

DeleteFlash(key string)
ClearFlashes()
```

The method names are self-explained. For example, if you need to get a message and remove it at the next request use the `GetFlash`. When you only need to retrieve but don't remove then use the `PeekFlash`.

> Flash messages are not stored in a database.

<!-- slide:break-80 -->
