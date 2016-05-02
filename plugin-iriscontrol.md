# Control panel

[This is a plugin](https://github.com/kataras/iris/tree/development/plugin/iriscontrol) which is working but not finished.

Which gives  access to your iris server's information via a web interface.
> You need internet connection the first time you will run this plugin, because the assets don't exists to this repository but [here](https://github.com/iris-contrib/iris-control-assets). The plugin will install these for you at the first run.

-----

How to use
```go
iriscontrol.Web(port int, authenticatedUsers map[string]string) iris.IPlugin
```

Example

```go
package main

import (
    "github.com/kataras/iris"
    "github.com/kataras/iris/plugin/iriscontrol"
)

func main() {

    iris.Plugin(iriscontrol.Web(9090, map[string]string{
        "irisusername1": "irispassword1",
        "irisusername2": "irispassowrd2",
    }))

    iris.Get("/", func(ctx *iris.Context) {
    })

    iris.Post("/something", func(ctx *iris.Context) {
    })

    iris.Listen()
}

```

-----



#Thanks for reading this e-book!

## To be continued...
