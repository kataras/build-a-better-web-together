# Plugins

Plugins are modules which get injected into Iris' flow. They're like middleware for the Iris framework itself. 
Middlewares start their actions after the server processes requests and get executed on every request, 
plugins on the other hand start working when you registered them. 
Plugins work with the framework's code, they have access to the *iris.Framework, so they are able register routes, 
start a second server, read Iris' configs or edit them and so on. 
The Plugin interface looks this:

```go
type (
	// Plugin just an empty base for plugins
	// A Plugin can be added with: .Add(PreListenFunc(func(*Framework))) and so on... or
	// .Add(myPlugin{},myPlugin2{}) myPlugin is a struct with any of the methods below or
	// .PostListen(func(*Framework)) and so on...
	Plugin interface {
	}

	// pluginGetName implements the GetName() string method
	pluginGetName interface {
		// GetName has to return the name of the plugin, a name is unique.
		// name has to be not dependent from other methods of the plugin,
		// because it is being called even before Activate()
		GetName() string
	}

	// pluginGetDescription implements the GetDescription() string method
	pluginGetDescription interface {
		// GetDescription has to return the description of what the plugins is used for
		GetDescription() string
	}

	// pluginActivate implements the Activate(pluginContainer) error method
	pluginActivate interface {
		// Activate called BEFORE the plugin being added to the plugins list,
		// if Activate() returns none nil error then the plugin is not being added to the list
		// it's called only once
		//
		// PluginContainer parameter used to add other plugins if that's necessary by the plugin
		Activate(PluginContainer) error
	}

	// pluginPreListen implements the PreListen(*Framework) method
	pluginPreListen interface {
		// PreListen is called only once, BEFORE the server is started (if .Listen called)
		//  parameter is the station
		PreListen(*Framework)
	}

	// PreListenFunc implements the simple function listener for the PreListen(*Framework)
	PreListenFunc func(*Framework)

	// pluginPostListen implements the PostListen(*Framework) method
	pluginPostListen interface {
		// PostListen is called once, AFTER the server is started (if .Listen called)
		// parameter is the station
		PostListen(*Framework)
	}

	// PostListenFunc implements the simple function listener for the PostListen(*Framework)
	PostListenFunc func(*Framework)

	// pluginPreClose implements the PreClose(*Framework) method
	pluginPreClose interface {
		// PreClose is called once, BEFORE the Iris.Close method
		// any plugin cleanup/clear memory happens here
		//
		// The plugin is deactivated after this state
		PreClose(*Framework)
	}

	// PreCloseFunc implements the simple function listener for the PreClose(*Framework)
	PreCloseFunc func(*Framework)

	// pluginPreDownload It's for the future, not being used, I need to create
	// and return an ActivatedPlugin type which will have it's methods, and pass it on .Activate
	// but now we return the whole pluginContainer, which I can't determinate which plugin tries to
	// download something, so we will leave it here for the future.
	pluginPreDownload interface {
		// PreDownload it's being called every time a plugin tries to download something
		//
		// first parameter is the plugin
		// second parameter is the download url
		// must return a boolean, if false then the plugin is not permmited to download this file
		PreDownload(plugin Plugin, downloadURL string) // bool
	}

	// PreDownloadFunc implements the simple function listener for the PreDownload(plugin,string)
	PreDownloadFunc func(Plugin, string)

)
```

```go
package main

import (
	"fmt"

	"github.com/kataras/iris"
)

func main() {
	// first way:
	// simple way for simple things
	// PreListen before a station is listening ( iris.Listen/TLS...)
	iris.Plugins.PreListen(func(s *iris.Framework) {
		for _, route := range s.Lookups() {
			fmt.Printf("Func: Route Method: %s | Subdomain %s | Path: %s is going to be registed with %d handler(s). \n", route.Method(), route.Subdomain(), route.Path(), len(route.Middleware()))
		}
	})

	// second way:
	// structured way for more things
	plugin := myPlugin{}
	iris.Plugins.Add(plugin)

	iris.Get("/first_route", aHandler)

	iris.Post("/second_route", aHandler)

	iris.Put("/third_route", aHandler)

	iris.Get("/fourth_route", aHandler)

	iris.Listen(":8080")
}

func aHandler(ctx *iris.Context) {
	ctx.Write("Hello from: %s", ctx.PathString())
}

type myPlugin struct{}

// PostListen after a station is listening (iris.Listen/TLS...)
func (pl myPlugin) PostListen(s *iris.Framework) {
	fmt.Printf("myPlugin: server is listening on host: %s", s.HTTPServer.Host())
}

//list:
/*
	Activate(iris.PluginContainer)
	GetName() string
	GetDescription() string
	PreListen(*iris.Framework)
	PostListen(*iris.Framework)
	PreClose(*iris.Framework)
	PreDownload(thePlugin iris.Plugin, downloadUrl string)
*/

```

An example of one plugin which is under development is Iris control, a web interface that gives you remote control to your Iris web server. 
You can find it's code [here](https://github.com/iris-contrib/plugin/tree/master/iriscontrol).

Take a look at [the plugin.go](https://github.com/iris-contrib/plugin), it's easy to make your own plugin.

Custom callbacks can be maden with third-party package [go-events](https://github.com/kataras/go-events).
