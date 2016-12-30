# Graceful

[This is a package](https://github.com/iris-contrib/graceful).


Enables graceful shutdown.

```go

package main

import (
	"time"
	"gopkg.in/kataras/iris.v5"
	"github.com/iris-contrib/graceful"
)

func main() {
	api := iris.New()
	api.Get("/", func(c *iris.Context) {
		c.Write("Welcome to the home page!")
	})

	graceful.Run(":3001", time.Duration(10)*time.Second, api)
}


```