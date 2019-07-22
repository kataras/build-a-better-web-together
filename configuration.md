# Configuration

At the [previous](host/) section we've learnt about the first input argument of the `app.Run`, here we will take a look of what the second one is.

Let's start from basics. The `iris.New` function returns an `iris.Application`. This Application value can be configured through its `Configure(...iris.Configurator)` and `Run` methods.

The second optional, varadiac argument of `app.Run` method accepts one or more `iris.Configurator`. An `iris.Configurator` is just a type of: `func(app *iris.Application)`. Custom `iris.Configurator` can be passed to modify yours `*iris.Application` as well.

There are built-in`iris.Configurator`s for each of the core [Configuration](https://godoc.org/github.com/kataras/iris#Configuration)'s fields, such as `iris.WithoutStartupLog`, `iris.WithCharset("UTF-8")`, `iris.WithOptimizations` and `iris.WithConfiguration(iris.Configuration{...})` functions.

Each “module” like the iris view engine, websockets, sessions and each middleware has its own configurations and options, most of them, separately from the core configuration.

## Using the [Configuration](https://godoc.org/github.com/kataras/iris#Configuration)

The only one configuration structure is the `iris.Configuration`. Let's start by that one which can be passed on the `iris.WithConfiguration` function to make it an `iris.Configurator`.

All of the `iris.Configuration` fields are defaulted to the most common use cases. Iris doesn’t need any configuration before its `app.Run` but if you want to make use of a custom `iris.Configurator` before the server runs then you can use the `app.Configure` method to pass the configurator\(s\) there.

```go
config := iris.WithConfiguration(iris.Configuration {
  DisableStartupLog: true,
  Optimizations: true,
  Charset: "UTF-8",
})

app.Run(iris.Addr(":8080"), config)
```

### Load from [YAML](https://yaml.org/)

Using the `iris.YAML("path")`.

File: **iris.yml**

```yaml
FireMethodNotAllowed: true
DisableBodyConsumptionOnUnmarshal: true
TimeFormat: Mon, 01 Jan 2006 15:04:05 GMT
Charset: UTF-8
```

File: **main.go**

```go
config := iris.WithConfiguration(iris.YAML("./iris.yml"))
app.Run(iris.Addr(":8080"), config)
```

### Load from [TOML](https://github.com/toml-lang/toml)

Using the `iris.TOML("path")`.

File: **iris.tml**

```text
FireMethodNotAllowed = true
DisableBodyConsumptionOnUnmarshal = false
TimeFormat = "Mon, 01 Jan 2006 15:04:05 GMT"
Charset = "UTF-8"

[Other]
    ServerName = "my fancy iris server"
    ServerOwner = "admin@example.com"
```

File: **main.go**

```go
config := iris.WithConfiguration(iris.TOML("./iris.tml"))
app.Run(iris.Addr(":8080"), config)
```

## Using the functional way

As we already mention, you can pass any number of `iris.Configurator` in the `app.Run`’s second argument. Iris provides an option for each of its `iris.Configuration`’s fields.

```go
app.Run(iris.Addr(":8080"), iris.WithoutInterruptHandler,
    iris.WithoutServerError(iris.ErrServerClosed),
    iris.WithoutBodyConsumptionOnUnmarshal,
    iris.WithoutAutoFireStatusCode,
    iris.WithOptimizations,
    iris.WithTimeFormat("Mon, 01 Jan 2006 15:04:05 GMT"),
)
```

Good when you want to change some of the configuration's field. Prefix: "With" or "Without", code editors will help you navigate through all configuration options without even a glitch to the documentation.

## Custom values

The `iris.Configuration` contains a field called `Other map[string]interface{}` which accepts any custom `key:value` option, therefore you can use that field to pass specific values that your app expects based on the custom requirements.

```go
app.Run(iris.Addr(":8080"), 
    iris.WithOtherValue("ServerName", "my amazing iris server"),
    iris.WithOtherValue("ServerOwner", "admin@example.com"),
)
```

You can access those fields via `app.ConfigurationReadOnly`.

```go
serverName := app.ConfigurationReadOnly().Other["MyServerName"]
serverOwner := app.ConfigurationReadOnly().Other["ServerOwner"]
```

## Access Configuration from Context

Inside a handler, retrieve those fields using the following:

```go
ctx.Application().ConfigurationReadOnly()
```

