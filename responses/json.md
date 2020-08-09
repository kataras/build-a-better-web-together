# JSON

**Content-Type: "application/json"**

The `Context.JSON(v, ...opts)` is the method which sends JSON responses to the client. It accepts the value and optional settings for rendering. The `JSON` options structure looks like this:

```go
// JSON contains the options for the JSON (Context's) Renderer.
type JSON struct {
	// http-specific
	StreamingJSON bool
	// content-specific
	UnescapeHTML bool
	Indent       string
	Prefix       string
    ASCII        bool
    // if true then it prepends a "while(1);" when Go slice (to JSON Array) value.
	Secure       bool
	// proto.Message specific marshal options.
	Proto ProtoMarshalOptions
}
```

> If `Indent` field is empty and the application runs without optimizations, the `Indent` field will be automatically set to `2 spaces`.

So, if we want to write a JSON with indentation of four spaces and prefixed with `while(1)` we can do something like that:

```go
func handler(ctx iris.Context) {
    response := map[string]interface{}{"message": "ok"}
    options := iris.JSON{Indent: "    ", Secure:true}
    ctx.JSON(response, options)
}
```

**Result**

```json
{
    "message": "ok"
}
```

As with all RESTful rich responses, any marshallable (JSON at this case) Go type can be given. If we want to render a Go struct as JSON, the struct's fields we want to render should be **[exported](https://tour.golang.org/basics/3)**, and optionally tagged with the `json` struct tag. Look the exaple below:

```go
type User struct {
    Firstname    string `json:"firstname"`
    Lastname     string `json:"lastname"`
    IgnoredField int    `json:"-"`
}

func handler(ctx iris.Context) {
    response := User{
        Firstname: "makis",
        Lastname: "maropoulos",
        IgnoredField:42,
    }

    ctx.JSON(response)
}
```

**Result**

```json
{
  "firstname": "makis",
  "lastname": "maropoulos"
}
```

<!-- slide:break-100 -->
