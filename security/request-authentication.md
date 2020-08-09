# Request Authentication

Iris offers request authentication through its [jwt middleware](https://github.com/iris-contrib/middleware/tree/master/jwt). In this chapter you will learn the basics of how to use JWT with Iris.

**1.** Install it by executing the following shell command:

```bash
$ go get github.com/iris-contrib/middleware/jwt
```

**2.** To create a new jwt middleware use the `jwt.New` function. This example extracts the token through a `"token"` url parameter. Authenticated clients should be designed to set that with a signed token. The default jwt middleware's behavior to extract a token value is by the `Authentication: Bearer $TOKEN` header.

The jwt middleware has three methods to validate tokens.

* The first one is the `Serve` method - it is an `iris.Handler`,
* the second one is the `CheckJWT(iris.Context) bool` and
* the third one is a helper to retrieve the validated token - the `Get(iris.Context) *jwt.Token`.

**3.** To register it you simply prepend the jwt `j.Serve` middleware to a specific group of routes, a single route, or globally e.g. `app.Get("/secured", j.Serve, myAuthenticatedHandler)`.

**4.** To generate a token inside a handler that accepts a user payload and responses the signed token which later on can be sent through client requests headers or url parameter in this case, e.g. `jwt.NewToken` or `jwt.NewTokenWithClaims`.

## Example

```go
import (
    "github.com/kataras/iris/v12"
    "github.com/iris-contrib/middleware/jwt"
)

func getTokenHandler(ctx iris.Context) {
    token := jwt.NewTokenWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
        "foo": "bar",
    })

    // Sign and get the complete encoded token as a string using the secret
    tokenString, _ := token.SignedString([]byte("My Secret"))

    ctx.HTML(`Token: ` + tokenString + `<br/><br/>
    <a href="/secured?token=` + tokenString + `">/secured?token=` + tokenString + `</a>`)
}

func myAuthenticatedHandler(ctx iris.Context) {
    user := ctx.Values().Get("jwt").(*jwt.Token)

    ctx.Writef("This is an authenticated request\n")
    ctx.Writef("Claim content:\n")

    foobar := user.Claims.(jwt.MapClaims)
    for key, value := range foobar {
        ctx.Writef("%s = %s", key, value)
    }
}

func main(){
    app := iris.New()

    j := jwt.New(jwt.Config{
        // Extract by "token" url parameter.
        Extractor: jwt.FromParameter("token"),

        ValidationKeyGetter: func(token *jwt.Token) (interface{}, error) {
            return []byte("My Secret"), nil
        },
        SigningMethod: jwt.SigningMethodHS256,
    })

    app.Get("/", getTokenHandler)
    app.Get("/secured", j.Serve, myAuthenticatedHandler)
    app.Listen(":8080")
}
```

You can use any payload as "claims".

<!-- slide:break-80 -->
