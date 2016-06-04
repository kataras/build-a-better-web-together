# Subdomains


Subdomains are splitted to two categories, first is the static subdomain and second is the dynamic static domain.

- static : when you know the subdomain, usage: `controlpanel.mydomain.com` 
- dynamic : when you don't know the subdomain, usage: `user1993.mydomain.com` 

Static subdomains in Iris are simple [Parties](party.md).

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

Dynamic subdomains also easy but you have to define them with other way
