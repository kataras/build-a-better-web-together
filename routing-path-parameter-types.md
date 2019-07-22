Iris has the easiest and the most powerful routing process you have ever met.

Iris has its own interpeter for route's path syntax, parsing and evaluation (yes, like a programming language!).


It's fast, how? It calculates its needs and if not any special regexp needed then it just
registers the route with the low-level path syntax,
otherwise it pre-compiles the regexp and adds the necessary middleware(s). That means that you have zero performance cost compared to other routers or web frameworks.

## Parameters

A path parameter's name should contain only alphabetical letters. Numbers or symbols like  '_' are NOT allowed.

Do not confuse `ctx.Params()` with `ctx.Values()`.

* Path parameter's values can be retrieved from `ctx.Params()`.
* Context's local storage that can be used to communicate between handlers and middleware(s) can be stored to `ctx.Values()`.

The built-in available parameter types can be found at the following table.

| Param Type | Go Type | Validation | Retrieve Helper |
| -----------------|------|-------------|------|
| `:string` | string | anything (single path segment) | `Params().Get` |
| `:int` | int | -9223372036854775808 to 9223372036854775807 (x64) or -2147483648 to 2147483647 (x32), depends on the host arch | `Params().GetInt` |
| `:int8` | int8 | -128 to 127 | `Params().GetInt8` |
| `:int16` | int16 | -32768 to 32767 | `Params().GetInt16` |
| `:int32` | int32 | -2147483648 to 2147483647 | `Params().GetInt32` |
| `:int64` | int64 | -9223372036854775808 to 9223372036854775807 | `Params().GetInt64` |
| `:uint` | uint | 0 to 18446744073709551615 (x64) or 0 to 4294967295 (x32), depends on the host arch | `Params().GetUint` |
| `:uint8` | uint8 | 0 to 255 | `Params().GetUint8` |
| `:uint16` | uint16 | 0 to 65535 | `Params().GetUint16` |
| `:uint32` | uint32 | 0 to 4294967295 | `Params().GetUint32` |
| `:uint64` | uint64 | 0 to 18446744073709551615 | `Params().GetUint64` |
| `:bool` | bool | "1" or "t" or "T" or "TRUE" or "true" or "True" or "0" or "f" or "F" or "FALSE" or "false" or "False" | `Params().GetBool` |
| `:alphabetical` | string | lowercase or uppercase letters | `Params().Get` |
| `:file` | string | lowercase or uppercase letters, numbers, underscore (_), dash (-), point (.) and no spaces or other special characters that are not valid for filenames | `Params().Get` |
| `:path` | string | anything, can be separated by slashes (path segments) but should be the last part of the route path | `Params().Get` | 

**Usage**:

```go
app.Get("/users/{id:uint64}", func(ctx iris.Context){
    id := ctx.Params().GetUint64Default("id", 0)
    // [...]
})
```

| Built-in Func | Param Types |
| -----------|---------------|
| `regexp`(expr string) | :string |
| `prefix`(prefix string) | :string |
| `suffix`(suffix string) | :string |
| `contains`(s string) | :string |
| `min`(minValue int or int8 or int16 or int32 or int64 or uint8 or uint16 or uint32 or uint64  or float32 or float64) | :string(char length), :int, :int8, :int16, :int32, :int64, :uint, :uint8, :uint16, :uint32, :uint64  |
| `max`(maxValue int or int8 or int16 or int32 or int64 or uint8 or uint16 or uint32 or uint64 or float32 or float64) | :string(char length), :int, :int8, :int16, :int32, :int64, :uint, :uint8, :uint16, :uint32, :uint64 |
| `range`(minValue, maxValue int or int8 or int16 or int32 or int64 or uint8 or uint16 or uint32 or uint64 or float32 or float64) | :int, :int8, :int16, :int32, :int64, :uint, :uint8, :uint16, :uint32, :uint64 |

**Usage**:

```go
app.Get("/profile/{name:alphabetical max(255)}", func(ctx iris.Context){
    name := ctx.Params().Get("name")
    // len(name) <=255 otherwise this route will fire 404 Not Found
    // and this handler will not be executed at all.
})
```

**Do It Yourself**:

The `RegisterFunc` can accept any function that returns a `func(paramValue string) bool`.
Or just a `func(string) bool`.
If the validation fails then it will fire `404` or whatever status code the `else` keyword has.

```go
latLonExpr := "^-?[0-9]{1,3}(?:\\.[0-9]{1,10})?$"
latLonRegex, _ := regexp.Compile(latLonExpr)

// Register your custom argument-less macro function to the :string param type.
// MatchString is a type of func(string) bool, so we use it as it is.
app.Macros().Get("string").RegisterFunc("coordinate", latLonRegex.MatchString)

app.Get("/coordinates/{lat:string coordinate()}/{lon:string coordinate()}",
func(ctx iris.Context) {
    ctx.Writef("Lat: %s | Lon: %s", ctx.Params().Get("lat"), ctx.Params().Get("lon"))
})
```

Register your custom macro function which accepts two int arguments.

```go

app.Macros().Get("string").RegisterFunc("range",
func(minLength, maxLength int) func(string) bool {
    return func(paramValue string) bool {
        return len(paramValue) >= minLength && len(paramValue) <= maxLength
    }
})

app.Get("/limitchar/{name:string range(1,200) else 400}", func(ctx iris.Context) {
    name := ctx.Params().Get("name")
    ctx.Writef(`Hello %s | the name should be between 1 and 200 characters length
    otherwise this handler will not be executed`, name)
})
```

Register your custom macro function which accepts a slice of strings `[...,...]`.

```go
app.Macros().Get("string").RegisterFunc("has",
func(validNames []string) func(string) bool {
    return func(paramValue string) bool {
        for _, validName := range validNames {
            if validName == paramValue {
                return true
            }
        }

        return false
    }
})

app.Get("/static_validation/{name:string has([kataras,maropoulos])}",
func(ctx iris.Context) {
    name := ctx.Params().Get("name")
    ctx.Writef(`Hello %s | the name should be "kataras" or "maropoulos"
    otherwise this handler will not be executed`, name)
})
```

**Example Code**:

```go
func main() {
    app := iris.Default()

    // This handler will match /user/john but will not match neither /user/ or /user.
    app.Get("/user/{name}", func(ctx iris.Context) {
        name := ctx.Params().Get("name")
        ctx.Writef("Hello %s", name)
    })

    // This handler will match /users/42
    // but will not match /users/-1 because uint should be bigger than zero
    // neither /users or /users/.
    app.Get("/users/{id:uint64}", func(ctx iris.Context) {
        id := ctx.Params().GetUint64Default("id", 0)
        ctx.Writef("User with ID: %d", id)
    })

    // However, this one will match /user/john/send and also /user/john/everything/else/here
    // but will not match /user/john neither /user/john/.
    app.Post("/user/{name:string}/{action:path}", func(ctx iris.Context) {
        name := ctx.Params().Get("name")
        action := ctx.Params().Get("action")
        message := name + " is " + action
        ctx.WriteString(message)
    })

    app.Run(iris.Addr(":8080"))
}
```

> When parameter type is missing then it defaults to the `string` one, therefore `{name:string}` and `{name}` refers to the same exactly thing.
