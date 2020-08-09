# Sessions

When you work with an application, you open it, do some changes, and then you close it. This is much like a Session. The computer knows who you are. It knows when you start the application and when you end. But on the internet there is one problem: the web server does not know who you are or what you do, because the HTTP address doesn't maintain state.

Session variables solve this problem by storing user information to be used across multiple pages (e.g. username, favorite color, etc). By default, session variables last until the user closes the browser.

So; Session variables hold information about one single user, and are available to all pages in one application.

> **Tip**: If you need a permanent storage, you may want to store the data in a [database](Sessions-database).

-------

Iris has its own session implementation and sessions manager live inside the [iris/sessions](https://github.com/kataras/iris/tree/master/sessions) subpackage. You'll need to import this package in order to work with HTTP Sessions.

A session is started with the `Start` function of the `Sessions` object which is created with the `New` package-level function. That function will return a `Session`.

Session variables are set with the `Session.Set` method and retrieved by the `Session.Get` and its related methods. To delete a single variable use the `Session.Delete`. To delete and invalidate the whole session use the `Session.Destroy` method.

<details>
<summary>See all methods</summary>

The sessions manager is created using the `New` package-level function.

```go
import "github.com/kataras/iris/v12/sessions"

sess := sessions.New(sessions.Config{Cookie: "cookieName", ...})
```

Where `Config` looks like that.

```go
SessionIDGenerator func(iris.Context) string

// Defaults to "irissessionid".
Cookie string

CookieSecureTLS bool

// Defaults to false.
AllowReclaim bool

// Defaults to nil.
Encode func(cookieName string, value interface{}) (string, error)
// Defaults to nil.
Decode func(cookieName string, cookieValue string, v interface{}) error
// Defaults to nil.
Encoding Encoding

// Defaults to infinitive/unlimited life duration(0).
Expires time.Duration

// Defaults to false.
DisableSubdomainPersistence bool
```

The return value a `Sessions` pointer exports the following methods.

```go
Start(ctx iris.Context,
    cookieOptions ...iris.CookieOption) *Session

Destroy()
DestroyAll()
DestroyByID(sessID string)
OnDestroy(callback func(sid string))

ShiftExpiration(ctx iris.Context,
    cookieOptions ...iris.CookieOption) error
UpdateExpiration(ctx iris.Context, expires time.Duration,
    cookieOptions ...iris.CookieOption) error

UseDatabase(db Database)
```

Where `CookieOption` is just a `func(*http.Cookie)` which allows to customize cookie's properties.

The `Start` method returns a `Session` pointer value which exports its own methods to work per session.

```go
func (ctx iris.Context) {
    session := sess.Start(ctx)
     .ID() string
     .IsNew() bool

     .Set(key string, value interface{})
     .SetImmutable(key string, value interface{})
     .GetAll() map[string]interface{}
     .Delete(key string) bool
     .Clear()

     .Get(key string) interface{}
     .GetString(key string) string
     .GetStringDefault(key string, defaultValue string) string
     .GetInt(key string) (int, error)
     .GetIntDefault(key string, defaultValue int) int
     .Increment(key string, n int) (newValue int)
     .Decrement(key string, n int) (newValue int)
     .GetInt64(key string) (int64, error)
     .GetInt64Default(key string, defaultValue int64) int64
     .GetFloat32(key string) (float32, error)
     .GetFloat32Default(key string, defaultValue float32) float32
     .GetFloat64(key string) (float64, error)
     .GetFloat64Default(key string, defaultValue float64) float64
     .GetBoolean(key string) (bool, error)
     .GetBooleanDefault(key string, defaultValue bool) bool

     .SetFlash(key string, value interface{})
     .HasFlash() bool
     .GetFlashes() map[string]interface{}

     .PeekFlash(key string) interface{}
     .GetFlash(key string) interface{}
     .GetFlashString(key string) string
     .GetFlashStringDefault(key string, defaultValue string) string

     .DeleteFlash(key string)
     .ClearFlashes()

     .Destroy()
}

```

</details>

## Example

In this example we will only allow authenticated users to view our secret message on the /secret age. To get access to it, the will first have to visit /login to get a valid session cookie, hich logs him in. Additionally he can visit /logout to revoke his access to our secret message.

```go
// sessions.go
package main

import (
    "github.com/kataras/iris/v12"

    "github.com/kataras/iris/v12/sessions"
)

var (
    cookieNameForSessionID = "mycookiesessionnameid"
    sess                   = sessions.New(sessions.Config{Cookie: cookieNameForSessionID})
)

func secret(ctx iris.Context) {
    // Check if user is authenticated
    if auth, _ := sess.Start(ctx).GetBoolean("authenticated"); !auth {
        ctx.StatusCode(iris.StatusForbidden)
        return
    }

    // Print secret message
    ctx.WriteString("The cake is a lie!")
}

func login(ctx iris.Context) {
    session := sess.Start(ctx)

    // Authentication goes here
    // ...

    // Set user as authenticated
    session.Set("authenticated", true)
}

func logout(ctx iris.Context) {
    session := sess.Start(ctx)

    // Revoke users authentication
    session.Set("authenticated", false)
    // Or to remove the variable:
    session.Delete("authenticated")
    // Or destroy the whole session:
    session.Destroy()
}

func main() {
    app := iris.New()

    app.Get("/secret", secret)
    app.Get("/login", login)
    app.Get("/logout", logout)

    app.Listen(":8080")
}
```

```sh
$ go run sessions.go

$ curl -s http://localhost:8080/secret
Forbidden

$ curl -s -I http://localhost:8080/login
Set-Cookie: mysessionid=MTQ4NzE5Mz...

$ curl -s --cookie "mysessionid=MTQ4NzE5Mz..." http://localhost:8080/secret
The cake is a lie!
```

## Midddleware

When you want to use the `Session` in the same request handler and life cycle(chain of handlers, middlewares), you can
optionally register it as a middleware and use the package-level `sessions.Get` to retrieve the stored-to-context session.

The `Sessions` struct value contains the `Handler` method which can be used to return an `iris.Handler` to be registered as middleware. 

```go
import "github.com/kataras/iris/v12/sessions"

sess := sessions.New(sessions.Config{...})

app := iris.New()
app.Use(sess.Handler())

app.Get("/path", func(ctx iris.Context){
    session := sessions.Get(ctx)
    // [use session...]
})
```

Also, if the sessions manager's `Config.AllowReclaim` is `true` then you can still call `sess.Start` as many times as you want in the same request life cycle without the need of registering it as a middleware.

Navigate to <https://github.com/kataras/iris/tree/master/_examples/sessions> for more examples about the sessions subpackage.

Continue by reading the [Flash Messages](Sessions-flash-messages) section.

<!-- slide:break-100 -->
