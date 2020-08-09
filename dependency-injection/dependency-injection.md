# Documentation

Iris provides first-class support for dependency injection through request handlers and server replies based on return value(s).

With Iris you get truly safe bindings. It is blazing-fast, near to raw handlers performance because we pre-allocates necessary information before the server even goes online!

A dependency can be either a function or a static value. A function dependency can accept a previously registered dependency as its input argument too.

Example Code:

```go
func printFromTo(from, to string) string { return "message" }

// [...]
app.ConfigureContainer(func (api *iris.APIContainer){
    api.Get("/{from}/{to}", printFromTo)
})
```

As you've seen above the `iris.Context` input argument is totally optional. Iris is smart enough to bind it as well without any hassle.

### Overview

The most common scenario from a route to handle is to:

* accept one or more path parameters and request data, a payload
* send back a response, a payload (JSON, XML,...)

The new Iris Dependency Injection feature is about **33.2% faster** than its predecessor on the above case. This drops down even more the performance cost between native handlers and handlers with dependencies. This reason itself brings us, with safety and performance-wise, to the new `Party.ConfigureContainer(builder ...func(*iris.APIContainer)) *APIContainer` method which returns methods such as `Handle(method, relativePath string, handlersFn ...interface{}) *Route` and `RegisterDependency`.

Look how clean your codebase can be when using Iris':

```go
package main

import "github.com/kataras/iris/v12"

type (
    testInput struct {
        Email string `json:"email"`
    }

    testOutput struct {
        ID   int    `json:"id"`
        Name string `json:"name"`
    }
)

func handler(id int, in testInput) testOutput {
    return testOutput{
        ID:   id,
        Name: in.Email,
    }
}

func main() {
    app := iris.New()
    app.ConfigureContainer(func(api *iris.APIContainer) {
        api.Post("/{id:int}", handler)
    })
    app.Listen(":5000", iris.WithOptimizations)
}
```

Your eyes don't lie you. You read well, no `ctx.ReadJSON(&v)` and `ctx.JSON(send)` neither `error` handling are presented. It is a huge relief but if you ever need, you still have the control over those, even errors from dependencies. Here is a quick list of the new `Party.ConfigureContainer`()'s fields and methods:

```go
// Container holds the DI Container of this Party featured Dependency Injection.
// Use it to manually convert functions or structs(controllers) to a Handler.
Container *hero.Container
```

```go
// OnError adds an error handler for this Party's DI Hero Container and its handlers (or controllers).
// The "errorHandler" handles any error may occurred and returned
// during dependencies injection of the Party's hero handlers or from the handlers themselves.
OnError(errorHandler func(iris.Context, error))
```

```go
// RegisterDependency adds a dependency.
// The value can be a single struct value or a function.
// Follow the rules:
// * <T> {structValue}
// * func(accepts <T>)                                 returns <D> or (<D>, error)
// * func(accepts iris.Context)                        returns <D> or (<D>, error)
//
// A Dependency can accept a previous registered dependency and return a new one or the same updated.
// * func(accepts1 <D>, accepts2 <T>)                  returns <E> or (<E>, error) or error
// * func(acceptsPathParameter1 string, id uint64)     returns <T> or (<T>, error)
//
// Usage:
//
// - RegisterDependency(loggerService{prefix: "dev"})
// - RegisterDependency(func(ctx iris.Context) User {...})
// - RegisterDependency(func(User) OtherResponse {...})
RegisterDependency(dependency interface{})

// UseResultHandler adds a result handler to the Container.
// A result handler can be used to inject the returned struct value
// from a request handler or to replace the default renderer.
UseResultHandler(handler func(next iris.ResultHandler) iris.ResultHandler)
```

```go
type ResultHandler func(ctx iris.Context, v interface{}) error
```

```go
// Use same as a common Party's "Use" but it accepts dynamic functions as its "handlersFn" input.
Use(handlersFn ...interface{})
// Done same as a common Party's but it accepts dynamic functions as its "handlersFn" input.
Done(handlersFn ...interface{})
```

```go
// Handle same as a common Party's `Handle` but it accepts one or more "handlersFn" functions which each one of them
// can accept any input arguments that match with the Party's registered Container's `Dependencies` and
// any output result; like custom structs <T>, string, []byte, int, error,
// a combination of the above, hero.Result(hero.View | hero.Response) and more.
//
// It's common from a hero handler to not even need to accept a `Context`, for that reason,
// the "handlersFn" will call `ctx.Next()` automatically when not called manually.
// To stop the execution and not continue to the next "handlersFn"
// the end-developer should output an error and return `iris.ErrStopExecution`.
Handle(method, relativePath string, handlersFn ...interface{}) *Route

// Get registers a GET route, same as `Handle("GET", relativePath, handlersFn....)`.
Get(relativePath string, handlersFn ...interface{}) *Route
// and so on...
```

There is a list of all available builtin dependencies:

> Reference: [Inputs](inputs.md)

### Request & Response & Path Parameters

**1.** Declare Go types for client's request body and a server's response.

```go
type (
    request struct {
        Firstname string `json:"firstname"`
        Lastname  string `json:"lastname"`
    }

    response struct {
        ID      uint64 `json:"id"`
        Message string `json:"message"`
    }
)
```

**2.** Create the route handler.

Path parameters and request body are binded automatically.

* **id uint64** binds to "id:uint64"
* **input request** binds to client request data such as JSON

```go
func updateUser(id uint64, input request) response {
    return response{
        ID:      id,
        Message: "User updated successfully",
    }
}
```

**3.** Configure the container per group and register the route.

```go
app.Party("/user").ConfigureContainer(container)

func container(api *iris.APIContainer) {
    api.Put("/{id:uint64}", updateUser)
}
```

**4.** Simulate a [client](https://curl.haxx.se/download.html) request which sends data to the server and displays the response.

```bash
curl --request PUT -d '{"firstanme":"John","lastname":"Doe"}' http://localhost:8080/user/42
```

```javascript
{
    "id": 42,
    "message": "User updated successfully"
}
```

### Custom Preflight

Before we continue to the next section, register dependencies, you may want to learn how a response can be customized through the `iris.Context` right before sent to the client.

The server will automatically execute the `Preflight(iris.Context) error` method of a function's output struct value right before send the response to the client.

Take for example that you want to fire different HTTP status codes depending on the custom logic inside your handler and also modify the value(response body) itself before sent to the client. Your response type should contain a `Preflight` method like below.

```go
type response struct {
    ID      uint64 `json:"id,omitempty"`
    Message string `json:"message"`
    Code    int    `json:"code"`
    Timestamp int64 `json:"timestamp,omitempty"`
}

func (r *response) Preflight(ctx iris.Context) error {
    if r.ID > 0 {
        r.Timestamp = time.Now().Unix()
    }

    ctx.StatusCode(r.Code)
    return nil
}
```

Now, each handler that returns a `*response` value will call the `response.Preflight` method automatically.

```go
func deleteUser(db *sql.DB, id uint64) *response {
    // [...custom logic]

    return &response{
        Message: "User has been marked for deletion",
        Code: iris.StatusAccepted,
    }
}
```

If you register the route and fire a request you should see an output like this, the timestamp is filled and the HTTP status code of the response that the client will receive is 202 (Status Accepted).

```javascript
{
  "message": "User has been marked for deletion",
  "code": 202,
  "timestamp": 1583313026
}
```

### Register Dependencies

**1.** Import packages to interact with a database. The go-sqlite3 package is a database driver for [SQLite](https://www.sqlite.org/index.html).

```go
import "database/sql"
import _ "github.com/mattn/go-sqlite3"
```

**2.** Configure the container ([see above](#request-and-response-and-path-parameters)), register your dependencies. Handler expects an \*sql.DB instance.

```go
localDB, _ := sql.Open("sqlite3", "./foo.db")
api.RegisterDependency(localDB)
```

**3.** Register a route to create a user.

```go
api.Post("/{id:uint64}", createUser)
```

**4.** The create user Handler.

The handler accepts a database and some client request data such as JSON, Protobuf, Form, URL Query and e.t.c. It Returns a response.

```go
func createUser(db *sql.DB, user request) *response {
    // [custom logic using the db]
    userID, err := db.CreateUser(user)
    if err != nil {
        return &response{
            Message: err.Error(),
            Code: iris.StatusInternalServerError,
        }
    }

    return &response{
        ID:      userID,
        Message: "User created",
        Code:    iris.StatusCreated,
    }
}
```

**5.** Simulate a [client](https://curl.haxx.se/download.html) to create a user.

```bash
# JSON
curl --request POST -d '{"firstname":"John","lastname":"Doe"}' \
--header 'Content-Type: application/json' \
http://localhost:8080/user
```

```bash
# Form (multipart)
curl --request POST 'http://localhost:8080/users' \
--header 'Content-Type: multipart/form-data' \
--form 'firstname=John' \
--form 'lastname=Doe'
```

```bash
# Form (URL-encoded)
curl --request POST 'http://localhost:8080/users' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'firstname=John' \
--data-urlencode 'lastname=Doe'
```

```bash
# URL Query
curl --request POST 'http://localhost:8080/users?firstname=John&lastname=Doe'
```

Response:

```javascript
{
    "id": 42,
    "message": "User created",
    "code": 201,
    "timestamp": 1583313026
}
```

## Return values

Read the [list](outputs.md) of all output values and possible results.

> Reference: [Outputs](outputs.md)

```go
type response struct {
    ID      uint64 `json:"id,omitempty"`
    Message string `json:"message"`
    Code    int    `json:"code"`
    Timestamp int64 `json:"timestamp,omitempty"`
}

func (r *response) Preflight(ctx iris.Context) error {
    if r.ID > 0 {
        r.Timestamp = time.Now().Unix()
    }

    ctx.StatusCode(r.Code)
    return nil
}

func deleteUser(db *sql.DB, id uint64) *response {
    // [...custom logic]

    return &response{
        Message: "User has been marked for deletion",
        Code: iris.StatusAccepted,
    }
}
```

Later on you'll see how this knowledge will help you to craft an application using the MVC architectural pattern which Iris provides wonderful API for it.

<!-- slide:break-80 -->
