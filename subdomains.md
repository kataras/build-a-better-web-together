# Subdomains

Subdomains in Iris are simple [Parties](party.md).

```go
package main

import (
	"github.com/kataras/iris"
)

func main() {
	// first the subdomains.
	admin := iris.Party("admin.yourhost.com")
	{
		//this will only success on admin.yourhost.com/hey
		admin.Get("/", func(c *iris.Context) {
			c.Write("Welcome to admin.yourhost.com")
		})
		//this will only success on admin.yourhost.com/hey2
		admin.Get("/hey", func(c *iris.Context) {
			c.Write("Hey from admin.yourhost.com")
		})
	}

	iris.Get("/hey", func(c *iris.Context) {
		c.Write("Hey from no-subdomain yourhost.com")
	})

	iris.Listen(":80")
}


```