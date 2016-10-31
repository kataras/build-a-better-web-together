# Configuration

All configurations have default values, things will work as you expected with `iris.New()` 

but if you want to customize iris then pass `iris.New(iris.Configuration{})` or use the options-func, example: `iris.New(iris.OptionCharset("UTF-8"))` 

`.New` **by configuration**
```go
import "gopkg.in/kataras/iris.v4"
//...

myConfig := iris.Configuration{Charset: "UTF-8", IsDevelopment:true, Sessions: iris.SessionsConfiguration{Cookie:"mycookie"}, Websocket: iris.WebsocketConfiguration{Endpoint: "/my_endpoint"}}
iris.New(myConfig)
```

`.New` **by options**

```go
import "gopkg.in/kataras/iris.v4"
//...

iris.New(iris.OptionCharset("UTF-8"), iris.OptionIsDevelopment(true), iris.OptionSessionsCookie("mycookie"), iris.OptionWebsocketEndpoint("/my_endpoint"))

// if you want to set configuration after the .New use the .Set: 
iris.Set(iris.OptionDisableBanner(true))
```


### List of available options

```go
// OptionDisablePathCorrection corrects and redirects the requested path to the registed path
// for example, if /home/ path is requested but no handler for this Route found,
// then the Router checks if /home handler exists, if yes,
// (permant)redirects the client to the correct path /home
//
// Default is false
OptionDisablePathCorrection(val bool)

// OptionDisablePathEscape when is false then its escapes the path, the named parameters (if any).
OptionDisablePathEscape(val bool)

// OptionDisableBanner outputs the iris banner at startup
//
// Default is false
OptionDisableBanner(val bool)

// OptionLoggerOut is the destination for output
//
// Default is os.Stdout
OptionLoggerOut(val io.Writer)

// OptionLoggerPreffix is the logger's prefix to write at beginning of each line
//
// Default is [IRIS]
OptionLoggerPreffix(val string)

// OptionProfilePath a the route path, set it to enable http pprof tool
// Default is empty, if you set it to a $path, these routes will handled:
OptionProfilePath(val string)

// OptionDisableTemplateEngines set to true to disable loading the default template engine (html/template) and disallow the use of iris.UseEngine
// Default is false
OptionDisableTemplateEngines(val bool)

// OptionIsDevelopment iris will act like a developer, for example
// If true then re-builds the templates on each request
// Default is false
OptionIsDevelopment(val bool)

// OptionTimeFormat time format for any kind of datetime parsing
OptionTimeFormat(val string)

// OptionCharset character encoding for various rendering
// used for templates and the rest of the responses(via serializer)
// Default is "UTF-8"
OptionCharset(val string)

// OptionGzip enables gzip compression on your Render actions, this includes any type of render, templates and pure/raw content
// If you don't want to enable it globaly, you could just use the third parameter on context.Render("myfileOrResponse", structBinding{}, iris.RenderOptions{"gzip": true})
// Default is false
OptionGzip(val bool)

// OptionOther are the custom, dynamic options, can be empty
// this fill used only by you to set any app's options you want
// for each of an Iris instance
OptionOther(val ...options.Options) //map[string]interface{}, options is github.com/kataras/go-options

// OptionSessionsCookie string, the session's client cookie name, for example: "qsessionid"
OptionSessionsCookie(val string)

// OptionSessionsDecodeCookie set it to true to decode the cookie key with base64 URLEncoding
// Defaults to false
OptionSessionsDecodeCookie(val bool)

// OptionSessionsExpires the duration of which the cookie must expires (created_time.Add(Expires)).
// If you want to delete the cookie when the browser closes, set it to -1 but in this case, the server side's session duration is up to GcDuration
//
// Default infinitive/unlimited life duration(0)
OptionSessionsExpires(val time.Duration)

// OptionSessionsCookieLength the length of the sessionid's cookie's value, let it to 0 if you don't want to change it
// Defaults to 32
OptionSessionsCookieLength(val int)

// OptionSessionsGcDuration every how much duration(GcDuration) the memory should be clear for unused cookies (GcDuration)
// for example: time.Duration(2)*time.Hour. it will check every 2 hours if cookie hasn't be used for 2 hours,
// deletes it from backend memory until the user comes back, then the session continue to work as it was
//
// Default 2 hours
OptionSessionsGcDuration(val time.Duration)

// OptionSessionsDisableSubdomainPersistence set it to true in order dissallow your q subdomains to have access to the session cookie
// defaults to false
OptionSessionsDisableSubdomainPersistence(val bool)

// OptionWebsocketWriteTimeout time allowed to write a message to the connection.
// Default value is 15 * time.Second
OptionWebsocketWriteTimeout(val time.Duration)

// OptionWebsocketPongTimeout allowed to read the next pong message from the connection
// Default value is 60 * time.Second
OptionWebsocketPongTimeout(val time.Duration)

// OptionWebsocketPingPeriod send ping messages to the connection with this period. Must be less than PongTimeout
// Default value is (PongTimeout * 9) / 10
OptionWebsocketPingPeriod(val time.Duration)

// OptionWebsocketMaxMessageSize max message size allowed from connection
// Default value is 1024
OptionWebsocketMaxMessageSize(val int64)

// OptionWebsocketBinaryMessages set it to true in order to denotes binary data messages instead of utf-8 text
// see https://github.com/kataras/iris/issues/387#issuecomment-243006022 for more
// defaults to false
OptionWebsocketBinaryMessages(val bool)

// OptionWebsocketEndpoint is the path which the websocket server will listen for clients/connections
// Default value is empty string, if you don't set it the Websocket server is disabled.
OptionWebsocketEndpoint(val string)

// OptionWebsocketReadBufferSize is the buffer size for the underline reader
OptionWebsocketReadBufferSize(val int)

// OptionWebsocketWriteBufferSize is the buffer size for the underline writer
OptionWebsocketWriteBufferSize(val int)
```go


### ServerConfiguration

Note: One iris instance can have and listening to many iris' Servers, one iris' Server can be used/passed to many iris instance, with that in mind, the server configuration has its own options to set.


```go
// examples:
iris.AddServer(iris.OptionServerCertFile("file.cert"),iris.OptionServerKeyFile("file.key"))
iris.ListenTo(iris.OptionServerReadBufferSize(42000))

// or 
iris.AddServer(iris.ServerConfiguration{ListeningAddr: "mydomain.com", CertFile: "file.cert", KeyFile: "file.key"})
iris.ListenTo(iris.ServerConfiguration{ReadBufferSize:42000, ListeningAddr: "mydomain.com"})

// both are valid
```

**List** of all Server's options:

```go
OptionServerListeningAddr(val string)

OptionServerCertFile(val string)

OptionServerKeyFile(val string)

// AutoTLS enable to get certifications from the Letsencrypt
// when this configuration field is true, the CertFile & KeyFile are empty, no need to provide a key.
//
// example: https://github.com/iris-contrib/examples/blob/4.0.0/letsencyrpt/main.go
OptionServerAutoTLS(val bool)

// Mode this is for unix only
OptionServerMode(val os.FileMode)
// OptionServerMaxRequestBodySize Maximum request body size.
//
// The server rejects requests with bodies exceeding this limit.
//
// By default request body size is 8MB.
OptionServerMaxRequestBodySize(val int)

// Per-connection buffer size for requests' reading.
// This also limits the maximum header size.
//
// Increase this buffer if your clients send multi-KB RequestURIs
// and/or multi-KB headers (for example, BIG cookies).
//
// Default buffer size is used if not set.
OptionServerReadBufferSize(val int)

// Per-connection buffer size for responses' writing.
//
// Default buffer size is used if not set.
OptionServerWriteBufferSize(val int)

// Maximum duration for reading the full request (including body).
//
// This also limits the maximum duration for idle keep-alive
// connections.
//
// By default request read timeout is unlimited.
OptionServerReadTimeout(val time.Duration)

// Maximum duration for writing the full response (including body).
//
// By default response write timeout is unlimited.
OptionServerWriteTimeout(val time.Duration)

// RedirectTo, defaults to empty, set it in order to override the station's handler and redirect all requests to this address which is of form(HOST:PORT or :PORT)
//
// NOTE: the http status is 'StatusMovedPermanently', means one-time-redirect(the browser remembers the new addr and goes to the new address without need to request something from this server
// which means that if you want to change this address you have to clear your browser's cache in order this to be able to change to the new addr.
//
// example: https://github.com/iris-contrib/examples/tree/4.0.0/multiserver_listening2
OptionServerRedirectTo(val string)

// OptionServerVirtual If this server is not really listens to a real host, it mostly used in order to achieve testing without system modifications
OptionServerVirtual(val bool)

// OptionServerVListeningAddr, can be used for both virtual = true or false,
// if it's setted to not empty, then the server's Host() will return this addr instead of the ListeningAddr.
// server's Host() is used inside global template helper funcs
// set it when you are sure you know what it does.
//
// Default is empty ""
OptionServerVListeningAddr(val string)

// OptionServerVScheme if setted to not empty value then all template's helper funcs prepends that as the url scheme instead of the real scheme
// server's .Scheme returns VScheme if  not empty && differs from real scheme
//
// Default is empty ""
OptionServerVScheme(val string)

// OptionServerName the server's name, defaults to "iris".
// You're free to change it, but I will trust you to don't, this is the only setting whose somebody, like me, can see if iris web framework is used
OptionServerName(val string)

```    

### The main Configuration
```go
type Configuration struct {
	// DisablePathCorrection corrects and redirects the requested path to the registed path
	// for example, if /home/ path is requested but no handler for this Route found,
	// then the Router checks if /home handler exists, if yes,
	// (permant)redirects the client to the correct path /home
	//
	// Default is false
	DisablePathCorrection bool

	// DisablePathEscape when is false then its escapes the path, the named parameters (if any).
	// Change to true it if you want something like this https://github.com/kataras/iris/issues/135 to work
	//
	// When do you need to Disable(true) it:
	// accepts parameters with slash '/'
	// Request: http://localhost:8080/details/Project%2FDelta
	// ctx.Param("project") returns the raw named parameter: Project%2FDelta
	// which you can escape it manually with net/url:
	// projectName, _ := url.QueryUnescape(c.Param("project").
	// Look here: https://github.com/kataras/iris/issues/135 for more
	//
	// Default is false
	DisablePathEscape bool

	// DisableBanner outputs the iris banner at startup
	//
	// Default is false
	DisableBanner bool

	// LoggerOut is the destination for output
	//
	// Default is os.Stdout
	LoggerOut io.Writer
	// LoggerPreffix is the logger's prefix to write at beginning of each line
	//
	// Default is [IRIS]
	LoggerPreffix string

	// ProfilePath a the route path, set it to enable http pprof tool
	// Default is empty, if you set it to a $path, these routes will handled:
	// $path/cmdline
	// $path/profile
	// $path/symbol
	// $path/goroutine
	// $path/heap
	// $path/threadcreate
	// $path/pprof/block
	// for example if '/debug/pprof'
	// http://yourdomain:PORT/debug/pprof/
	// http://yourdomain:PORT/debug/pprof/cmdline
	// http://yourdomain:PORT/debug/pprof/profile
	// http://yourdomain:PORT/debug/pprof/symbol
	// http://yourdomain:PORT/debug/pprof/goroutine
	// http://yourdomain:PORT/debug/pprof/heap
	// http://yourdomain:PORT/debug/pprof/threadcreate
	// http://yourdomain:PORT/debug/pprof/pprof/block
	// it can be a subdomain also, for example, if 'debug.'
	// http://debug.yourdomain:PORT/
	// http://debug.yourdomain:PORT/cmdline
	// http://debug.yourdomain:PORT/profile
	// http://debug.yourdomain:PORT/symbol
	// http://debug.yourdomain:PORT/goroutine
	// http://debug.yourdomain:PORT/heap
	// http://debug.yourdomain:PORT/threadcreate
	// http://debug.yourdomain:PORT/pprof/block
	ProfilePath string

	// DisableTemplateEngines set to true to disable loading the default template engine (html/template) and disallow the use of iris.UseEngine
	// default is false
	DisableTemplateEngines bool

	// IsDevelopment iris will act like a developer, for example
	// If true then re-builds the templates on each request
	// default is false
	IsDevelopment bool

	// TimeFormat time format for any kind of datetime parsing
	TimeFormat string

	// Charset character encoding for various rendering
	// used for templates and the rest of the responses
	// defaults to "UTF-8"
	Charset string

	// Gzip enables gzip compression on your Render actions, this includes any type of render, templates and pure/raw content
	// If you don't want to enable it globaly, you could just use the third parameter on context.Render("myfileOrResponse", structBinding{}, iris.RenderOptions{"gzip": true})
	// defaults to false
	Gzip bool

	// Sessions contains the configs for sessions
	Sessions SessionsConfiguration

	// Websocket contains the configs for Websocket's server integration
	Websocket WebsocketConfiguration

	// Other are the custom, dynamic options, can be empty
	// this fill used only by you to set any app's options you want
	// for each of an Iris instance
	Other options.Options
}
```

View all configuration fields and options by navigating to the [kataras/iris/configuration.go source file](https://github.com/kataras/iris/blob/4.0.0/configuration.go), to view the testing configuration move [here](https://github.com/kataras/iris/blob/4.0.0/httptest/httptest.go)