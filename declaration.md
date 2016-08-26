# Declaration

You might have asked yourself:

* Q: Other frameworks need more lines to start a server, why is Iris different?
* A: Iris gives you the freedom to choose between three ways to use Iris

  1. global **iris.**
  2. declare a new iris station with default config: **iris.New\(\)** 
  3. declare a new iris station with custom config: ** api := iris.New\(config.Iris{...}\)**


Config can change after declaration with 1&2. `iris.Config.` 3. \/ `api.Config.`

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
```

Before looking at the 3rd way, let's take a quick look at the [**config**](configuration.md)**\*\***.Iris\*\*:

```go
type (
    // Iris configs for the station
    Iris struct {

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

        // Charset character encoding for various rendering
        // used for templates and the rest of the responses
        // defaults to "UTF-8"
        Charset string

        // Gzip enables gzip compression on your Render actions, this includes any type of render, templates and pure/raw content
        // If you don't want to enable it globaly, you could just use the third parameter on context.Render("myfileOrResponse", structBinding{}, iris.RenderOptions{"gzip": true})
        // defaults to false
        Gzip bool

        // Sessions contains the configs for sessions
        Sessions Sessions

        // Websocket contains the configs for Websocket's server integration
        Websocket *Websocket

        // Tester contains the configs for the test framework, so far we have only one because all test framework's configs are setted by the iris itself
        // You can find example on the https://github.com/kataras/iris/glob/master/context_test.go
        Tester Tester
    }
)
```

```go
// 3.
package main 

import (
  "github.com/kataras/iris"
  "github.com/kataras/iris/config"
)

func main() {
    c := config.Iris{
        ProfilePath:        "/mypath/debug",
    }
    // to get the default: c := config.Default()

    api := iris.New(c)
    api.Listen(":8080")
}

```

> Note that with 2. & 3. you **can define and Listen with more than one Iris server** in the
> same app, when it's necessary.

For profiling there are eight \(8\) generated routes with pages filled with info:

* \/mypath\/debug\/
* \/mypath\/debug\/cmdline
* \/mypath\/debug\/profile
* \/mypath\/debug\/symbol
* \/mypath\/debug\/goroutine
* \/mypath\/debug\/heap
* \/mypath\/debug\/threadcreate
* \/mypath\/debug\/pprof\/block

* More about configuration [here](configuration.md)


