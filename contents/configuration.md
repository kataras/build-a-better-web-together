# Configuration

At the [Host](host.md) section we've learnt about the first input argument of the `app.Run`, here we will take a look of what the second one is.

Let's start from basics. The `iris.New` function returns an `iris.Application`. This Application value can be configured through its `Configure(...iris.Configurator)` and `Run` methods.

The second optional, varadiac argument of `app.Run/Listen` method accepts one or more `iris.Configurator`. An `iris.Configurator` is just a type of: `func(app *iris.Application)`. Custom `iris.Configurator` can be passed to modify yours `*iris.Application` as well.

There are built-in`iris.Configurator`s for each of the core [Configuration](https://godoc.org/github.com/kataras/iris#Configuration)'s fields, such as `iris.WithoutStartupLog`, `iris.WithCharset("UTF-8")`, `iris.WithOptimizations` and `iris.WithConfiguration(iris.Configuration{...})` functions.

Each "module" like the iris view engine, websockets, sessions and each middleware has its own configurations and options.

## Using the [Configuration](https://godoc.org/github.com/kataras/iris#Configuration)

All of the `iris.Configuration` fields are defaulted to the most common use cases. If you want to make use of a custom `iris.Configurator` at any point call the `app.Configure(accepts iris.Configurator)` method to pass your configurator(s) there.

```go
config := iris.WithConfiguration(iris.Configuration {
  DisableStartupLog: true,
  Optimizations: true,
  Charset: "UTF-8",
})

app.Listen(":8080", config)
```

List of all available settings:

```go
// Tunnel is the Tunnels field of the TunnelingConfiguration structure.
type Tunnel struct {
    // Name is the only one required field,
    // it is used to create and close tunnels, e.g. "MyApp".
    // If this field is not empty then ngrok tunnels will be created
    // when the iris app is up and running.
    Name string `json:"name" yaml:"Name" toml:"Name"`
    // Addr is basically optionally as it will be set through
    // Iris built-in Runners, however, if `iris.Raw` is used
    // then this field should be set of form 'hostname:port'
    // because framework cannot be aware
    // of the address you used to run the server on this custom runner.
    Addr string `json:"addr,omitempty" yaml:"Addr" toml:"Addr"`
}

// TunnelingConfiguration contains configuration
// for the optional tunneling through ngrok feature.
// Note that the ngrok should be already installed at the host machine.
type TunnelingConfiguration struct {
    // AuthToken field is optionally and can be used
    // to authenticate the ngrok access.
    // ngrok authtoken <YOUR_AUTHTOKEN>
    AuthToken string `json:"authToken,omitempty" yaml:"AuthToken" toml:"AuthToken"`

    // No...
    // Config is optionally and can be used
    // to load ngrok configuration from file system path.
    //
    // If you don't specify a location for a configuration file,
    // ngrok tries to read one from the default location $HOME/.ngrok2/ngrok.yml.
    // The configuration file is optional; no error is emitted if that path does not exist.
    // Config string `json:"config,omitempty" yaml:"Config" toml:"Config"`

    // Bin is the system binary path of the ngrok executable file.
    // If it's empty then the framework will try to find it through system env variables.
    Bin string `json:"bin,omitempty" yaml:"Bin" toml:"Bin"`

    // WebUIAddr is the web interface address of an already-running ngrok instance.
    // Iris will try to fetch the default web interface address(http://127.0.0.1:4040)
    // to determinate if a ngrok instance is running before try to start it manually.
    // However if a custom web interface address is used,
    // this field must be set e.g. http://127.0.0.1:5050.
    WebInterface string `json:"webInterface,omitempty" yaml:"WebInterface" toml:"WebInterface"`

    // Region is optionally, can be used to set the region which defaults to "us".
    // Available values are:
    // "us" for United States
    // "eu" for Europe
    // "ap" for Asia/Pacific
    // "au" for Australia
    // "sa" for South America
    // "jp" forJapan
    // "in" for India
    Region string `json:"region,omitempty" yaml:"Region" toml:"Region"`

    // Tunnels the collection of the tunnels.
    // One tunnel per Iris Host per Application, usually you only need one.
    Tunnels []Tunnel `json:"tunnels" yaml:"Tunnels" toml:"Tunnels"`
}

// Configuration holds the necessary settings for an Iris Application instance.
// All fields are optionally, the default values will work for a common web application.
//
// A Configuration value can be passed through `WithConfiguration` Configurator.
// Usage:
// conf := iris.Configuration{ ... }
// app := iris.New()
// app.Configure(iris.WithConfiguration(conf)) OR
// app.Run/Listen(..., iris.WithConfiguration(conf)).
type Configuration struct {
    // LogLevel is the log level the application should use to output messages.
    // Logger, by default, is mostly used on Build state but it is also possible
    // that debug error messages could be thrown when the app is running, e.g.
    // when malformed data structures try to be sent on Client (i.e Context.JSON/JSONP/XML...).
    //
    // Defaults to "info". Possible values are:
    // * "disable"
    // * "fatal"
    // * "error"
    // * "warn"
    // * "info"
    // * "debug"
    LogLevel string `json:"logLevel" yaml:"LogLevel" toml:"LogLevel" env:"LOG_LEVEL"`

    // Tunneling can be optionally set to enable ngrok http(s) tunneling for this Iris app instance.
    // See the `WithTunneling` Configurator too.
    Tunneling TunnelingConfiguration `json:"tunneling,omitempty" yaml:"Tunneling" toml:"Tunneling"`

    // IgnoreServerErrors will cause to ignore the matched "errors"
    // from the main application's `Run` function.
    // This is a slice of string, not a slice of error
    // users can register these errors using yaml or toml configuration file
    // like the rest of the configuration fields.
    //
    // See `WithoutServerError(...)` function too.
    //
    // Example: https://github.com/kataras/iris/tree/master/_examples/http-server/listen-addr/omit-server-errors
    //
    // Defaults to an empty slice.
    IgnoreServerErrors []string `json:"ignoreServerErrors,omitempty" yaml:"IgnoreServerErrors" toml:"IgnoreServerErrors"`

    // DisableStartupLog if set to true then it turns off the write banner on server startup.
    //
    // Defaults to false.
    DisableStartupLog bool `json:"disableStartupLog,omitempty" yaml:"DisableStartupLog" toml:"DisableStartupLog"`
    // DisableInterruptHandler if set to true then it disables the automatic graceful server shutdown
    // when control/cmd+C pressed.
    // Turn this to true if you're planning to handle this by your own via a custom host.Task.
    //
    // Defaults to false.
    DisableInterruptHandler bool `json:"disableInterruptHandler,omitempty" yaml:"DisableInterruptHandler" toml:"DisableInterruptHandler"`

    // DisablePathCorrection disables the correcting
    // and redirecting or executing directly the handler of
    // the requested path to the registered path
    // for example, if /home/ path is requested but no handler for this Route found,
    // then the Router checks if /home handler exists, if yes,
    // (permanent)redirects the client to the correct path /home.
    //
    // See `DisablePathCorrectionRedirection` to enable direct handler execution instead of redirection.
    //
    // Defaults to false.
    DisablePathCorrection bool `json:"disablePathCorrection,omitempty" yaml:"DisablePathCorrection" toml:"DisablePathCorrection"`
    // DisablePathCorrectionRedirection works whenever configuration.DisablePathCorrection is set to false
    // and if DisablePathCorrectionRedirection set to true then it will fire the handler of the matching route without
    // the trailing slash ("/") instead of send a redirection status.
    //
    // Defaults to false.
    DisablePathCorrectionRedirection bool `json:"disablePathCorrectionRedirection,omitempty" yaml:"DisablePathCorrectionRedirection" toml:"DisablePathCorrectionRedirection"`
    // EnablePathIntelligence if set to true,
    // the router will redirect HTTP "GET" not found pages to the most closest one path(if any). For example
    // you register a route at "/contact" path -
    // a client tries to reach it by "/cont", the path will be automatic fixed
    // and the client will be redirected to the "/contact" path
    // instead of getting a 404 not found response back.
    //
    // Defaults to false.
    EnablePathIntelligence bool `json:"enablePathIntelligence,omitempty" yaml:"EnablePathIntelligence" toml:"EnablePathIntelligence"`
    // EnablePathEscape when is true then its escapes the path and the named parameters (if any).
    // When do you need to Disable(false) it:
    // accepts parameters with slash '/'
    // Request: http://localhost:8080/details/Project%2FDelta
    // ctx.Param("project") returns the raw named parameter: Project%2FDelta
    // which you can escape it manually with net/url:
    // projectName, _ := url.QueryUnescape(c.Param("project").
    //
    // Defaults to false.
    EnablePathEscape bool `json:"enablePathEscape,omitempty" yaml:"EnablePathEscape" toml:"EnablePathEscape"`
    // ForceLowercaseRouting if enabled, converts all registered routes paths to lowercase
    // and it does lowercase the request path too for matching.
    //
    // Defaults to false.
    ForceLowercaseRouting bool `json:"forceLowercaseRouting,omitempty" yaml:"ForceLowercaseRouting" toml:"ForceLowercaseRouting"`
    // FireMethodNotAllowed if it's true router checks for StatusMethodNotAllowed(405) and
    //  fires the 405 error instead of 404
    // Defaults to false.
    FireMethodNotAllowed bool `json:"fireMethodNotAllowed,omitempty" yaml:"FireMethodNotAllowed" toml:"FireMethodNotAllowed"`
    // DisableAutoFireStatusCode if true then it turns off the http error status code
    // handler automatic execution on error code from a `Context.StatusCode` call.
    // By-default a custom http error handler will be fired when "Context.StatusCode(errorCode)" called.
    //
    // Defaults to false.
    DisableAutoFireStatusCode bool `json:"disableAutoFireStatusCode,omitempty" yaml:"DisableAutoFireStatusCode" toml:"DisableAutoFireStatusCode"`
    // ResetOnFireErrorCode if true then any previously response body or headers through
    // response recorder or gzip writer will be ignored and the router
    // will fire the registered (or default) HTTP error handler instead.
    // See `core/router/handler#FireErrorCode` and `Context.EndRequest` for more details.
    //
    // Read more at: https://github.com/kataras/iris/issues/1531
    //
    // Defaults to false.
    ResetOnFireErrorCode bool `json:"resetOnFireErrorCode,omitempty" yaml:"ResetOnFireErrorCode" toml:"ResetOnFireErrorCode"`

    // EnableOptimization when this field is true
    // then the application tries to optimize for the best performance where is possible.
    //
    // Defaults to false.
    EnableOptimizations bool `json:"enableOptimizations,omitempty" yaml:"EnableOptimizations" toml:"EnableOptimizations"`
    // DisableBodyConsumptionOnUnmarshal manages the reading behavior of the context's body readers/binders.
    // If set to true then it
    // disables the body consumption by the `context.UnmarshalBody/ReadJSON/ReadXML`.
    //
    // By-default io.ReadAll` is used to read the body from the `context.Request.Body which is an `io.ReadCloser`,
    // if this field set to true then a new buffer will be created to read from and the request body.
    // The body will not be changed and existing data before the
    // context.UnmarshalBody/ReadJSON/ReadXML will be not consumed.
    DisableBodyConsumptionOnUnmarshal bool `json:"disableBodyConsumptionOnUnmarshal,omitempty" yaml:"DisableBodyConsumptionOnUnmarshal" toml:"DisableBodyConsumptionOnUnmarshal"`
    // FireEmptyFormError returns if set to tue true then the `context.ReadBody/ReadForm`
    // will return an `iris.ErrEmptyForm` on empty request form data.
    FireEmptyFormError bool `json:"fireEmptyFormError,omitempty" yaml:"FireEmptyFormError" yaml:"FireEmptyFormError"`

    // TimeFormat time format for any kind of datetime parsing
    // Defaults to  "Mon, 02 Jan 2006 15:04:05 GMT".
    TimeFormat string `json:"timeFormat,omitempty" yaml:"TimeFormat" toml:"TimeFormat"`

    // Charset character encoding for various rendering
    // used for templates and the rest of the responses
    // Defaults to "utf-8".
    Charset string `json:"charset,omitempty" yaml:"Charset" toml:"Charset"`

    // PostMaxMemory sets the maximum post data size
    // that a client can send to the server, this differs
    // from the overral request body size which can be modified
    // by the `context#SetMaxRequestBodySize` or `iris#LimitRequestBodySize`.
    //
    // Defaults to 32MB or 32 << 20 if you prefer.
    PostMaxMemory int64 `json:"postMaxMemory" yaml:"PostMaxMemory" toml:"PostMaxMemory"`
    //  +----------------------------------------------------+
    //  | Context's keys for values used on various featuers |
    //  +----------------------------------------------------+

    // Context values' keys for various features.
    //
    // LocaleContextKey is used by i18n to get the current request's locale, which contains a translate function too.
    //
    // Defaults to "iris.locale".
    LocaleContextKey string `json:"localeContextKey,omitempty" yaml:"LocaleContextKey" toml:"LocaleContextKey"`
    // LanguageContextKey is the context key which a language can be modified by a middleware.
    // It has the highest priority over the rest and if it is empty then it is ignored,
    // if it set to a static string of "default" or to the default language's code
    // then the rest of the language extractors will not be called at all and
    // the default language will be set instead.
    //
    // Use with `Context.SetLanguage("el-GR")`.
    //
    // See `i18n.ExtractFunc` for a more organised way of the same feature.
    // Defaults to "iris.locale.language".
    LanguageContextKey string `json:"languageContextKey,omitempty" yaml:"LanguageContextKey" toml:"LanguageContextKey"`
    // VersionContextKey is the context key which an API Version can be modified
    // via a middleware through `SetVersion` method, e.g. `ctx.SetVersion("1.0, 1.1")`.
    // Defaults to "iris.api.version".
    VersionContextKey string `json:"versionContextKey" yaml:"VersionContextKey" toml:"VersionContextKey"`
    // GetViewLayoutContextKey is the key of the context's user values' key
    // which is being used to set the template
    // layout from a middleware or the main handler.
    // Overrides the parent's or the configuration's.
    //
    // Defaults to "iris.ViewLayout"
    ViewLayoutContextKey string `json:"viewLayoutContextKey,omitempty" yaml:"ViewLayoutContextKey" toml:"ViewLayoutContextKey"`
    // GetViewDataContextKey is the key of the context's user values' key
    // which is being used to set the template
    // binding data from a middleware or the main handler.
    //
    // Defaults to "iris.viewData"
    ViewDataContextKey string `json:"viewDataContextKey,omitempty" yaml:"ViewDataContextKey" toml:"ViewDataContextKey"`
    // RemoteAddrHeaders are the allowed request headers names
    // that can be valid to parse the client's IP based on.
    // By-default no "X-" header is consired safe to be used for retrieving the
    // client's IP address, because those headers can manually change by
    // the client. But sometimes are useful e.g., when behind a proxy
    // you want to enable the "X-Forwarded-For" or when cloudflare
    // you want to enable the "CF-Connecting-IP", inneed you
    // can allow the `ctx.RemoteAddr()` to use any header
    // that the client may sent.
    //
    // Defaults to an empty map but an example usage is:
    // RemoteAddrHeaders {
    //    "X-Real-Ip":             true,
    //  "X-Forwarded-For":       true,
    //     "CF-Connecting-IP":      true,
    //    }
    //
    // Look `context.RemoteAddr()` for more.
    RemoteAddrHeaders map[string]bool `json:"remoteAddrHeaders,omitempty" yaml:"RemoteAddrHeaders" toml:"RemoteAddrHeaders"`
    // RemoteAddrPrivateSubnets defines the private sub-networks.
    // They are used to be compared against
    // IP Addresses fetched through `RemoteAddrHeaders` or `Context.Request.RemoteAddr`.
    // For details please navigate through: https://github.com/kataras/iris/issues/1453
    // Defaults to:
    // {
    //     Start: net.ParseIP("10.0.0.0"),
    //     End:   net.ParseIP("10.255.255.255"),
    // },
    // {
    //     Start: net.ParseIP("100.64.0.0"),
    //     End:   net.ParseIP("100.127.255.255"),
    // },
    // {
    //     Start: net.ParseIP("172.16.0.0"),
    //     End:   net.ParseIP("172.31.255.255"),
    // },
    // {
    //     Start: net.ParseIP("192.0.0.0"),
    //     End:   net.ParseIP("192.0.0.255"),
    // },
    // {
    //     Start: net.ParseIP("192.168.0.0"),
    //     End:   net.ParseIP("192.168.255.255"),
    // },
    // {
    //     Start: net.ParseIP("198.18.0.0"),
    //     End:   net.ParseIP("198.19.255.255"),
    // }
    //
    // Look `Context.RemoteAddr()` for more.
    RemoteAddrPrivateSubnets []netutil.IPRange `json:"remoteAddrPrivateSubnets" yaml:"RemoteAddrPrivateSubnets" toml:"RemoteAddrPrivateSubnets"`
    // SSLProxyHeaders defines the set of header key values
    // that would indicate a valid https Request (look `Context.IsSSL()`).
    // Example: `map[string]string{"X-Forwarded-Proto": "https"}`.
    //
    // Defaults to empty map.
    SSLProxyHeaders map[string]string `json:"sslProxyHeaders" yaml:"SSLProxyHeaders" toml:"SSLProxyHeaders"`
    // HostProxyHeaders defines the set of headers that may hold a proxied hostname value for the clients.
    // Look `Context.Host()` for more.
    // Defaults to empty map.
    HostProxyHeaders map[string]bool `json:"hostProxyHeaders" yaml:"HostProxyHeaders" toml:"HostProxyHeaders"`
    // Other are the custom, dynamic options, can be empty.
    // This field used only by you to set any app's options you want.
    //
    // Defaults to empty map.
    Other map[string]interface{} `json:"other,omitempty" yaml:"Other" toml:"Other"`
}
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
app.Listen(":8080", config)
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
app.Listen(":8080", config)
```

## Using the functional way

As we already mention, you can pass any number of `iris.Configurator` in the `app.Run/Listen`’s second argument. Iris provides an option for each of its `iris.Configuration`’s fields.

```go
app.Listen(":8080", iris.WithoutInterruptHandler,
    iris.WithoutBodyConsumptionOnUnmarshal,
    iris.WithoutAutoFireStatusCode,
    iris.WithLowercaseRouting,
    iris.WithPathIntelligence,
    iris.WithOptimizations,
    iris.WithTimeFormat("Mon, 01 Jan 2006 15:04:05 GMT"),
)
```

Good when you want to change some of the configuration's field. Prefix: "With" or "Without", code editors will help you navigate through all configuration options without even a glitch to the documentation.

List of functional options:

```go
var (
    WithGlobalConfiguration Configurator
    WithoutStartupLog, WithoutBanner Configurator
    WithoutInterruptHandler Configurator
    WithoutPathCorrection Configurator
    WithPathIntelligence Configurator
    WithoutPathCorrectionRedirection Configurator
    WithoutBodyConsumptionOnUnmarshal Configurator
    WithEmptyFormError Configurator
    WithPathEscape Configurator
    WithLowercaseRouting Configurator
    WithOptimizations Configurator
    WithFireMethodNotAllowed Configurator
    WithoutAutoFireStatusCode Configurator
    WithResetOnFireErrorCode Configurator
    WithTunneling Configurator
)

func WithLogLevel(level string) Configurator
func WithoutServerError(errors ...error) Configurator
func WithTimeFormat(timeformat string) Configurator
func WithCharset(charset string) Configurator
func WithPostMaxMemory(limit int64) Configurator
func WithRemoteAddrHeader(header ...string) Configurator
func WithoutRemoteAddrHeader(headerName string) Configurator
func WithRemoteAddrPrivateSubnet(startIP, endIP string) Configurator
func WithSSLProxyHeader(headerKey, headerValue string) Configurator
func WithHostProxyHeader(headers ...string) Configurator
func WithOtherValue(key string, val interface{}) Configurator
func WithSitemap(startURL string) Configurator
```

## Custom values

The `iris.Configuration` contains a field named `Other map[string]interface{}` which accepts any custom `key:value` option, therefore you can use that field to pass specific values that your app expects based on the custom requirements.

```go
app.Listen(":8080", 
    iris.WithOtherValue("ServerName", "my amazing iris server"),
    iris.WithOtherValue("ServerOwner", "admin@example.com"),
)
```

You can access those fields via `app.ConfigurationReadOnly`.

## Access Configuration from Context

```go
func (ctx iris.Context) {
    cfg := app.Application().ConfigurationReadOnly().GetOther()
    srvName := cfg["MyServerName"]
    srvOwner := cfg["ServerOwner"]
}
```

