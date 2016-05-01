# Plugins

Plugins are modules that you can build to inject the Iris' flow. Think it like a middleware for the Iris framework itself, not only the requests. Middleware starts it's actions after the server listen, Plugin on the other hand starts working when you registed them, from the begin, to the end. Look how it's interface looks:

```go
	// IPluginGetName implements the GetName() string method
	IPluginGetName interface {
		// GetName has to returns the name of the plugin, a name is unique
		// name has to be not dependent from other methods of the plugin,
		// because it is being called even before the Activate
		GetName() string
	}

	// IPluginGetDescription implements the GetDescription() string method
	IPluginGetDescription interface {
		// GetDescription has to returns the description of what the plugins is used for
		GetDescription() string
	}

	// IPluginGetDescription implements the Activate(IPluginContainer) error method
	IPluginActivate interface {
		// Activate called BEFORE the plugin being added to the plugins list,
		// if Activate returns none nil error then the plugin is not being added to the list
		// it is being called only one time
		//
		// PluginContainer parameter used to add other plugins if that's necessary by the plugin
		Activate(IPluginContainer) error
	}

	// IPluginPreHandle implements the PreHandle(IRoute) method
	IPluginPreHandle interface {
		// PreHandle it's being called every time BEFORE a Route is registed to the Router
		//
		//  parameter is the Route
		PreHandle(IRoute)
	}
	// IPluginPostHandle implements the PostHandle(IRoute) method
	IPluginPostHandle interface {
		// PostHandle it's being called every time AFTER a Route successfully registed to the Router
		//
		// parameter is the Route
		PostHandle(IRoute)
	}
	// IPluginPreListen implements the PreListen(*Station) method
	IPluginPreListen interface {
		// PreListen it's being called only one time, BEFORE the Server is started (if .Listen called)
		// is used to do work at the time all other things are ready to go
		//  parameter is the station
		PreListen(*Station)
	}
	// IPluginPostListen implements the PostListen(*Station) method
	IPluginPostListen interface {
		// PostListen it's being called only one time, AFTER the Server is started (if .Listen called)
		// parameter is the station
		PostListen(*Station)
	}
	// IPluginPreClose implements the PreClose(*Station) method
	IPluginPreClose interface {
		// PreClose it's being called only one time, BEFORE the Iris .Close method
		// any plugin cleanup/clear memory happens here
		//
		// The plugin is deactivated after this state
		PreClose(*Station)
	}
```

A small example, imagine that you want to get all routes registered to your server (OR modify them at runtime),  with their time registed, methods, (sub)domain  and the path, what whould you do on other frameworks when you want something from the framework which it doesn't supports out of the box? and what you can do with Iris:

```go
//file myplugin.go
package main

import (
	"time"

	"github.com/kataras/iris"
)

type RouteInfo struct {
	Method       string
	Domain       string
	Path         string
	TimeRegisted time.Time
}

type myPlugin struct {
	routes    []RouteInfo
}

func NewMyPlugin() *myPlugin {
	return &myPlugin{routes: make([]RouteInfo, 0)}
}

//
// Implement our plugin, you can view your inject points - listeners on the /kataras/iris/plugin.go too.
//
// Implement the PostHandle, because this is what we need now, we need to collect the information after a route is registed to our server so we do:
func (i *myPlugin) PostHandle(route iris.IRoute) {
	myRouteInfo := &RouteInfo{}
	myRouteInfo.Method = route.GetMethod()
	myRouteInfo.Domain = route.GetDomain()
	myRouteInfo.Path = route.GetPath()

	myRouteInfo.TimeRegisted = time.Now()

	i.routes = append(i.routes, myRouteInfo)
}

// PostListen called after the server is started, here you can do a lot of staff
// you have the right to access the whole iris' Station also, here you can add more routes and do anything you want, for example start a second server too, an admin web interface!
// for example let's print to the server's stdout the routes we collected...
func (i *myPlugin) PostListen(s *iris.Station) {
	s.Logger.Printf("From MyPlugin: You have registed %d routes ", len(i.routes))
	//do what ever you want, you have imagination do more than this!
}

//

```
Let's register our plugin:
```go

//file main.go
package main

import "github.com/kataras/iris"

func main() {
   iris.Plugins().Add(NewMyPlugin())
   //the plugin is running and saves all these routes
   iris.Get("/", func(c *iris.Context){})
   iris.Post("/login", func(c *iris.Context){})
   iris.Get("/login", func(c *iris.Context){})
   iris.Get("/something", func(c *iris.Context){})

   iris.Listen(":8080")
}


```
Output:

>From MyPlugin: You have registed 4 routes

An example of one plugin which is under development is the Iris control, a web interface that gives you control to your server remotely. You can find it's code [here](https://github.com/kataras/iris/tree/development/plugins/iriscontrol)
