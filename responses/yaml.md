# Markdown

**Content-Type: "application/x-yaml"**

**YAML** (a recursive acronym for "YAML Ain't Markup Language") is a human-readable data-serialization language. It is commonly used for **configuration** files and in applications where data is being stored or transmitted. YAML targets many of the same communications applications as Extensible Markup Language (XML) but has a minimal syntax which intentionally differs from [SGML](https://en.wikipedia.org/wiki/Standard_Generalized_Markup_Language).

The `Context.YAML(v)` is the method which sends YAML responses to the client. It accepts a value of any type. You only need the `yaml` struct field and all fields should be exported.

```go
type ExampleYAML struct {
	Name       string `yaml:"name"`
	ServerAddr string `yaml:"ServerAddr"`
}

func handler(ctx iris.Context) {
    response := ExampleYAML{Name: "Iris", ServerAddr: "localhost:8080"}
    ctx.YAML(response)
}
```

**Result**

```yaml
name: Iris
ServerAddr: localhost:8080
```

The same result can be achieved using `iris.Map` or a standard Go `map`:

```go
func handler(ctx iris.Context) {
    response := iris.Map{"name": "Iris", "serverAddr": "localhost:8080"}
    ctx.YAML(response)
}
```

References:
- [Wikipedia](https://en.wikipedia.org/wiki/YAML)
- [The Official YAML Web Site](https://yaml.org/)
