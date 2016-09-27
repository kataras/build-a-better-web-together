# Declaration

You might have asked yourself:

* Q: Other frameworks need more lines to start a server, why is Iris different?
* A: Iris gives you the freedom to choose between four ways to use Iris

  1. global **iris.**
  2. declare a new iris station with default config: **iris.New\(\)** 
  3. declare a new iris station with custom config: ** api := iris.New\(iris.Configuration{...}\)**
  4. declare a new iris station with custom options: ** api := iris.New\(iris.OptionCharset\("UTF-8"\), iris.OptionSessionsCookie\("mycookie"\), ...\)**


Configuration is **OPTIONAL  **and can change after declaration with`$instance.Config.`, \/ `$instance.Set(Option...)`

```go
import "github.com/kataras/iris"

// 1.
func firstWay() {

    iris.Get("/home",func(c *iris.Context){})
    iris.Listen(":8080")
}
// 2.
func secondWay() {

    api := iris.New()
    api.Get("/home",func(c *iris.Context){})
    api.Listen(":8080")
}

// 3.
func thirdWay() {

   config := iris.Configuration{IsDevelopment: true}
   iris.New(config)
   iris.Get("/home", func(c*iris.Context){})
   iris.Listen(":8080")
}

// 4.
func forthWay() {

    api := iris.New()
    api.Set(iris.OptionCharset("UTF-8"))

    api.Get("/home",func(c *iris.Context){})
    api.Listen(":8080")
}

// after .New, at runtime, also possible because Iris have default values, configuration is TOTALLY OPTIONAL DSESIRE
func main() {
    iris.Config.Websocket.Endpoind = "/ws"

   //...

   iris.Listen(":8080")
}
```



Let's take a quick look at the [**iris.Configuration**](configuration.md)

```go
// Configuration the whole configuration for an iris instance ($instance.Config) or global iris instance (iris.Config)
// these can be passed via options also, look at the top of this file(configuration.go)
//
// Configuration is also implements the OptionSet so it's a valid option itself, this is briliant enough
type Configuration struct {
    // VHost is the addr or the domain that server listens to, which it's optional
    // When to set VHost manually:
    // 1. it's automatically setted when you're calling
    //     $instance.Listen/ListenUNIX/ListenTLS/ListenLETSENCRYPT functions or
    //     ln,_ := iris.TCP4/UNIX/TLS/LETSENCRYPT; $instance.Serve(ln)
    // 2. If you using a balancer, or something like nginx
    //    then set it in order to have the correct url
    //    when calling the template helper '{{url }}'
    //    *keep note that you can use {{urlpath }}) instead*
    //
    // Note: this is the main's server Host, you can setup unlimited number of fasthttp servers
    // listening to the $instance.Handler after the manually-called $instance.Build
    //
    // Default comes from iris.Listen/.Serve with iris' listeners (iris.TCP4/UNIX/TLS/LETSENCRYPT)
    VHost string

    // VScheme is the scheme (http:// or https://) putted at the template function '{{url }}'
    // It's an optional field,
    // When to set VScheme manually:
    // 1. You didn't start the main server using $instance.Listen/ListenTLS/ListenLETSENCRYPT or $instance.Serve($instance.TCP4()/.TLS...)
    // 2. if you're using something like nginx and have iris listening with addr only(http://) but the nginx mapper is listening to https://
    //
    // Default comes from iris.Listen/.Serve with iris' listeners (TCP4,UNIX,TLS,LETSENCRYPT)
    VScheme string

    // MaxRequestBodySize Maximum request body size.
    //
    // The server rejects requests with bodies exceeding this limit.
    //
    // By default request body size is 8MB.
    MaxRequestBodySize int

    // Per-connection buffer size for requests' reading.
    // This also limits the maximum header size.
    //
    // Increase this buffer if your clients send multi-KB RequestURIs
    // and/or multi-KB headers (for example, BIG cookies).
    //
    // Default buffer size is used if not set.
    ReadBufferSize int

    // Per-connection buffer size for responses' writing.
    //
    // Default buffer size is used if not set.
    WriteBufferSize int

    // Maximum duration for reading the full request (including body).
    //
    // This also limits the maximum duration for idle keep-alive
    // connections.
    //
    // By default request read timeout is unlimited.
    ReadTimeout time.Duration

    // Maximum duration for writing the full response (including body).
    //
    // By default response write timeout is unlimited.
    WriteTimeout time.Duration

    // Maximum number of concurrent client connections allowed per IP.
    //
    // By default unlimited number of concurrent connections
    MaxConnsPerIP int

    // Maximum number of requests served per connection.
    //
    // The server closes connection after the last request.
    // 'Connection: close' header is added to the last response.
    //
    // By default unlimited number of requests may be served per connection.
    MaxRequestsPerConn int

    // CheckForUpdates will try to search for newer version of Iris based on the https://github.com/kataras/iris/releases
    // If a newer version found then the app will ask the he dev/user if want to update the 'x' version
    // if 'y' is pressed then the updater will try to install the latest version
    // the updater, will notify the dev/user that the update is finished and should restart the App manually.
    // Notes:
    // 1. Experimental feature
    // 2. If setted to true, the app will start the server normally and runs the updater in its own goroutine,
    //    for a sync operation see CheckForUpdatesSync.
    // 3. If you as developer edited the $GOPATH/src/github/kataras or any other Iris' Go dependencies at the past
    //    then the update process will fail.
    //
    // Usage: iris.Set(iris.OptionCheckForUpdates(true)) or
    //        iris.Config.CheckForUpdates = true or
    //        app := iris.New(iris.OptionCheckForUpdates(true))
    // Default is false
    CheckForUpdates bool
    // CheckForUpdatesSync checks for updates before server starts, it will have a little delay depends on the machine's download's speed
    // See CheckForUpdates for more
    // Notes:
    // 1. you could use the CheckForUpdatesSync while CheckForUpdates is false, set this or CheckForUpdates to true not both
    // 2. if both CheckForUpdates and CheckForUpdatesSync are setted to true then the updater will run in sync mode, before server server starts.
    //
    // Default is false
    CheckForUpdatesSync bool

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
    Sessions SessionsConfiguration {
        // Cookie string, the session's client cookie name, for example: "mysessionid"
        //
        // Defaults to "gosessionid"
        Cookie string

        // DecodeCookie set it to true to decode the cookie key with base64 URLEncoding
        //
        // Defaults to false
        DecodeCookie bool

        // Expires the duration of which the cookie must expires (created_time.Add(Expires)).
        // If you want to delete the cookie when the browser closes, set it to -1 but in this case, the server side's session duration is up to GcDuration
        //
        // Defaults to infinitive/unlimited life duration(0)
        Expires time.Duration

        // CookieLength the length of the sessionid's cookie's value, let it to 0 if you don't want to change it
        //
        // Defaults to 32
        CookieLength int

        // GcDuration every how much duration(GcDuration) the memory should be clear for unused cookies (GcDuration)
        // for example: time.Duration(2)*time.Hour. it will check every 2 hours if cookie hasn't be used for 2 hours,
        // deletes it from backend memory until the user comes back, then the session continue to work as it was
        //
        // Defaults to 2 hours
        GcDuration time.Duration

        // DisableSubdomainPersistence set it to true in order dissallow your q subdomains to have access to the session cookie
        //
        // Defaults to false
        DisableSubdomainPersistence bool

        // DisableAutoGC disables the auto-gc running when the session manager is created
        // set to false to disable this behavior, you can call(onnce) manually the GC using the .GC function
        //
        // Defaults to false
        DisableAutoGC bool
    }

    // Websocket contains the configs for Websocket's server integration
    Websocket WebsocketConfiguration {
        // WriteTimeout time allowed to write a message to the connection.
        // Default value is 15 * time.Second
        WriteTimeout time.Duration
        // PongTimeout allowed to read the next pong message from the connection
        // Default value is 60 * time.Second
        PongTimeout time.Duration
        // PingPeriod send ping messages to the connection with this period. Must be less than PongTimeout
        // Default value is (PongTimeout * 9) / 10
        PingPeriod time.Duration
        // MaxMessageSize max message size allowed from connection
        // Default value is 1024
        MaxMessageSize int64
        // BinaryMessages set it to true in order to denotes binary data messages instead of utf-8 text
        // see https://github.com/kataras/iris/issues/387#issuecomment-243006022 for more
        // defaults to false
        BinaryMessages bool
        // Endpoint is the path which the websocket server will listen for clients/connections
        // Default value is empty string, if you don't set it the Websocket server is disabled.
        Endpoint string
        // ReadBufferSize is the buffer size for the underline reader
        ReadBufferSize int
        // WriteBufferSize is the buffer size for the underline writer
        WriteBufferSize int
    }

    // Tester contains the configs for the test framework, so far we have only one because all test framework's configs are setted by the iris itself
    // You can find example on the https://github.com/kataras/iris/glob/master/context_test.go
    Tester TesterConfiguration {
        // ExplicitURL If true then the url (should) be prepended manually, useful when want to test subdomains
        // Default is false
        ExplicitURL bool
        // Debug if true then debug messages from the httpexpect will be shown when a test runs
        // Default is false
        Debug bool
    }

    // Other are the custom, dynamic options, can be empty
    // this fill used only by you to set any app's options you want
    // for each of an Iris instance
    Other options.Options
}


```

> Note that with 2.,  3. & 4. you **can serve more than one Iris server** in the
> same app, when it's necessary.

`.New` **by configuration**

```go

import "github.com/kataras/iris"

//...



myConfig := iris.Configuration{Charset: "UTF-8", IsDevelopment:true, Sessions: iris.SessionsConfiguration{Cookie:"mycookie"}, Websocket: iris.WebsocketConfiguration{Endpoint: "/my_endpoint"}}

iris.New(myConfig)

```

`.New` **by options**

```go

import "github.com/kataras/iris"

//...



iris.New(iris.OptionCharset("UTF-8"), iris.OptionIsDevelopment(true), 
iris.OptionSessionsCookie("mycookie"), iris.OptionWebsocketEndpoint("/my_endpoint"))



// if you want to set configuration after the .New use the .Set:

iris.Set(iris.OptionDisableBanner(true))

```

