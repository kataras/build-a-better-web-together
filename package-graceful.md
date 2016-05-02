# Graceful

[This is a middleware](https://githu)
Enables graceful shutdown.



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