# Declaration

Let's make a pause,

- Q: Other frameworks needs more lines to start a server, why Iris is different?
- A: Iris gives you the freedom to choose between three ways to declare to use Iris

 1. global **iris.**
 2. declare a new iris station with default config: **iris.New()** 
 3. declare a new iris station with custom config: ** api := iris.New(config.Iris{...})**
 
all of their configs can change after declaration with 1&2. `iris.Config().` 3. / `api.Config().` 



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

Before 3rd way, let's take a quick look at the **[config](configuration.md).Iris**:
```go
// Iris configs for the station
	// All fields can be changed before server's listen except the PathCorrection field
	//
	// MaxRequestBodySize is the only options that can be changed after server listen -
	// using Config().MaxRequestBodySize = ...
	// Render's rest config can be changed after declaration but before server's listen -
	// using Config().Render.Rest...
	// Render's Template config can be changed after declaration but before server's listen -
	// using Config().Render.Template...
	// Sessions config can be changed after declaration but before server's listen -
	// using Config().Sessions...
	// and so on...
	Iris struct {
		// MaxRequestBodySize Maximum request body size.
		//
		// The server rejects requests with bodies exceeding this limit.
		//
		// By default request body size is unlimited.
		MaxRequestBodySize int
		// PathCorrection corrects and redirects the requested path to the registed path
		// for example, if /home/ path is requested but no handler for this Route found,
		// then the Router checks if /home handler exists, if yes,
		// (permant)redirects the client to the correct path /home
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

		// Sessions the config for sessions
		// contains 3(three) properties
		// Provider: (look /sessions/providers)
		// Secret: cookie's name (string)
		// Life: cookie life (time.Duration)
		Sessions Sessions

		// Render contains the configs for template and rest configuration
		Render Render
	}
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
		Profile:            true,
		ProfilePath:        "/mypath/debug",
	}
    // to get the default: c := config.Default()
    
	api := iris.New(c)
	api.Listen(":8080")
}

```

> Note that with 2. & 3. you **can define and Listen to more than one Iris station** in the
> same app, when it's necessary.



For profiling  there are eight (8) generated routes with filed pages:

 -   /debug/pprof
 -  /debug/pprof/cmdline
 -  /debug/pprof/profile
 -  /debug/pprof/symbol
 -  /debug/pprof/goroutine
 -  /debug/pprof/heap
 -  /debug/pprof/threadcreate
 -  /debug/pprof/pprof/block


**PathCorrection**
corrects and redirects the requested path to the registered path
for example, if /home/ path is requested but no handler for this Route found,
then the Router checks if /home handler exists, if yes, redirects the client to the correct path /home
and VICE - VERSA if /home/ is registered but /home is requested then it redirects to /home/ (Default is true)

-  More about configuration [here](configuration.md)