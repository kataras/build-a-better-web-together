# Declaration

Let's make a pause,

- Q: Other frameworks needs more lines to start a server, why Iris is different?
- A: Iris gives you the freedom to choose between three ways to declare to use Iris

 1. global **iris.**
 2. declare a new iris station with default config: **iris.New()**
 3. declare a new iris station with custom config: **iris.New(iris.IrisConfig{...})**



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

Before 3rd way, let's take a quick look at the **iris.IrisConfig**:
```go
IrisConfig struct {
		// MaxRequestBodySize Maximum request body size.
		//
		// The server rejects requests with bodies exceeding this limit.
		//
		// By default request body size is unlimited.
		MaxRequestBodySize int
		// PathCorrection corrects and redirects the requested path to the registed path
		// for example, if /home/ path is requested but no handler for this Route found,
		// then the Router checks if /home handler exists, if yes, redirects the client to the correct path /home
		// and VICE - VERSA if /home/ is registed but /home is requested then it redirects to /home/
		//
		// Default is true
		PathCorrection bool

		// Log turn it to false if you want to disable logger,
		// Iris prints/logs ONLY errors, so be careful when you disable it
		Log bool

		// Profile set to true to enable web pprof (debug profiling)
		// Default is false, enabling makes available these 7 routes:
		// /debug/pprof/cmdline
		// /debug/pprof/profile
		// /debug/pprof/symbol
		// /debug/pprof/goroutine
		// /debug/pprof/heap
		// /debug/pprof/threadcreate
		// /debug/pprof/pprof/block
		Profile bool

		// ProfilePath change it if you want other url path than the default
		// Default is /debug/pprof , which means yourhost.com/debug/pprof
		ProfilePath string
		// Render specify configs for rendering
		Render iris.RenderConfig
	}

```
```go
// 3.
func thirdMethod() {

	config := &IrisConfig{
		PathCorrection:     true,
		MaxRequestBodySize: -1,
		Log:                true,
		Profile:            false,
		ProfilePath:        DefaultProfilePath,
		Render: &RenderConfig{
			Directory:                 "templates",
			Asset:                     nil,
			AssetNames:                nil,
			Layout:                    "",
			Extensions:                []string{".html"},
			Funcs:                     []template.FuncMap{},
			Delims:                    Delims{"{{", "}}"},
			Charset:                   DefaultCharset,
			IndentJSON:                false,
			IndentXML:                 false,
			PrefixJSON:                []byte(""),
			PrefixXML:                 []byte(""),
			HTMLContentType:           "text/html",
			IsDevelopment:             false,
			UnEscapeHTML:              false,
			StreamingJSON:             false,
			RequirePartials:           false,
			DisableHTTPErrorRendering: false,
		}}//these are the default values that you can change
	// DefaultProfilePath = "/debug/pprof"
	// DefaultCharset = "UTF-8"

	api := iris.New(config)
	api.Get("/home",func(c *iris.Context){})
	api.Listen(":8080")
}

```

> Note that with 2. & 3. you **can define and use more than one Iris station** in the
> same app, when it's necessary.

As you can see there are some options that you can chage at your iris declaration.

For example if we do that...
```go
package main

import "github.com/kataras/iris"

func main() {
	config := iris.IrisConfig{
		Profile:            true,
		ProfilePath:        "/mypath/debug",
	}

	api := iris.New(config)
	api.Listen(":8080")
}
```
run it, then you can open your browser, type '**localhost:8080/mypath/debug/profile**' at the location input field and you should see a webpage  shows you informations about CPU.

For profiling & debug there are seven (7) generated pages ('/debug/pprof/' is the default profile path, which on previous example we changed it to '/mypath/debug'):

 1. /debug/pprof/cmdline
 2. /debug/pprof/profile
 3. /debug/pprof/symbol
 4. /debug/pprof/goroutine
 5. /debug/pprof/heap
 6. /debug/pprof/threadcreate
 7. /debug/pprof/pprof/block


**PathCorrection**
corrects and redirects the requested path to the registed path
for example, if /home/ path is requested but no handler for this Route found,
then the Router checks if /home handler exists, if yes, redirects the client to the correct path /home
and VICE - VERSA if /home/ is registed but /home is requested then it redirects to /home/ (Default is true)