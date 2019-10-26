# Testing

Iris offers an incredible support for the [httpexpect](https://github.com/gavv/httpexpect), a Testing Framework for web applications. The `iris/httptest` subpackage provides helpers for Iris + httpexpect.

if you prefer the Go's standard [net/http/httptest](https://golang.org/pkg/net/http/httptest/) package, you can still use it. Iris as much every http web framework is compatible with any external tool for testing, at the end it's HTTP.

## Basic Authentication

In our first example we will use the `iris/httptest` to test Basic Authentication.

**1.** The `main.go` source file looks like that:

```go
package main

import (
    "github.com/kataras/iris/v12"
    "github.com/kataras/iris/v12/middleware/basicauth"
)

func newApp() *iris.Application {
    app := iris.New()

    authConfig := basicauth.Config{
        Users: map[string]string{"myusername": "mypassword"},
    }

    authentication := basicauth.New(authConfig)

    app.Get("/", func(ctx iris.Context) { ctx.Redirect("/admin") })

    needAuth := app.Party("/admin", authentication)
    {
        //http://localhost:8080/admin
        needAuth.Get("/", h)
        // http://localhost:8080/admin/profile
        needAuth.Get("/profile", h)

        // http://localhost:8080/admin/settings
        needAuth.Get("/settings", h)
    }

    return app
}

func h(ctx iris.Context) {
    username, password, _ := ctx.Request().BasicAuth()
    // third parameter  ^ will be always true because the middleware
    // makes sure for that, otherwise this handler will not be executed.

    ctx.Writef("%s %s:%s", ctx.Path(), username, password)
}

func main() {
    app := newApp()
    app.Run(iris.Addr(":8080"))
}
```

**2.** Now, create a `main_test.go` file and copy-paste the following.

```go
package main

import (
    "testing"

    "github.com/kataras/iris/v12/httptest"
)

func TestNewApp(t *testing.T) {
    app := newApp()
    e := httptest.New(t, app)

    // redirects to /admin without basic auth
    e.GET("/").Expect().Status(httptest.StatusUnauthorized)
    // without basic auth
    e.GET("/admin").Expect().Status(httptest.StatusUnauthorized)

    // with valid basic auth
    e.GET("/admin").WithBasicAuth("myusername", "mypassword").Expect().
        Status(httptest.StatusOK).Body().Equal("/admin myusername:mypassword")
    e.GET("/admin/profile").WithBasicAuth("myusername", "mypassword").Expect().
        Status(httptest.StatusOK).Body().Equal("/admin/profile myusername:mypassword")
    e.GET("/admin/settings").WithBasicAuth("myusername", "mypassword").Expect().
        Status(httptest.StatusOK).Body().Equal("/admin/settings myusername:mypassword")

    // with invalid basic auth
    e.GET("/admin/settings").WithBasicAuth("invalidusername", "invalidpassword").
        Expect().Status(httptest.StatusUnauthorized)

}
```

**3.** Open your command line and execute:

```bash
$ go test -v
```

## Other example: cookies

```go
package main

import (
    "fmt"
    "testing"

    "github.com/kataras/iris/v12/httptest"
)

func TestCookiesBasic(t *testing.T) {
    app := newApp()
    e := httptest.New(t, app, httptest.URL("http://example.com"))

    cookieName, cookieValue := "my_cookie_name", "my_cookie_value"

    // Test Set A Cookie.
    t1 := e.GET(fmt.Sprintf("/cookies/%s/%s", cookieName, cookieValue)).
        Expect().Status(httptest.StatusOK)
    // Validate cookie's existence, it should be available now.
    t1.Cookie(cookieName).Value().Equal(cookieValue)
    t1.Body().Contains(cookieValue)

    path := fmt.Sprintf("/cookies/%s", cookieName)

    // Test Retrieve A Cookie.
    t2 := e.GET(path).Expect().Status(httptest.StatusOK)
    t2.Body().Equal(cookieValue)

    // Test Remove A Cookie.
    t3 := e.DELETE(path).Expect().Status(httptest.StatusOK)
    t3.Body().Contains(cookieName)

    t4 := e.GET(path).Expect().Status(httptest.StatusOK)
    t4.Cookies().Empty()
    t4.Body().Empty()
}
```

```bash
$ go test -v -run=TestCookiesBasic$
```

Iris web framework itself uses this package to test itself. In the [_examples repository directory](https://github.com/kataras/iris/tree/master/_examples) you will find some useful tests as well. For more information please take a look and read the [httpexpect's documentation](https://github.com/gavv/httpexpect).

