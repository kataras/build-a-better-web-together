# Graceful

Graceful package is not part of the Iris, it's not a Middleware neither a Plugin, so a new repository created,
which it's a fork of [https://github.com/tylerb/graceful](https://github.com/tylerb/graceful).



How to use:

```go

package main

import (
	"time"

	"github.com/iris-contrib/graceful"
	"github.com/kataras/iris"
)

func main() {
	api := iris.New()
	api.Get("/", func(c *iris.Context) {
		c.Write("Welcome to the home page!")
	})

	graceful.Run(":3001", time.Duration(10)*time.Second, api)
}


```