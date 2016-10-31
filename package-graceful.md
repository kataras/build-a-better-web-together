# Graceful

[This is a package](https://github.com/iris-contrib/graceful/tree/4.0.0).


Enables graceful shutdown.

```go

package main

import (
	"time"
	"gopkg.in/kataras/iris.v4"
	"gopkg.in/iris-contrib/graceful.v4"
)

func main() {
	api := iris.New()
	api.Get("/", func(c *iris.Context) {
		c.Write("Welcome to the home page!")
	})

	graceful.Run(":3001", time.Duration(10)*time.Second, api)
}


```