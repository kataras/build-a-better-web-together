# Subdomains


Subdomains are split into two categories, first is the static subdomain and second is the dynamic subdomain.

- static : when you know the subdomain, usage: `controlpanel.mydomain.com` 
- dynamic : when you don't know the subdomain, usage: `user1993.mydomain.com`, `otheruser.mydomain.com` 

Iris has the simplest known form for subdomains, simple as [Parties](party.md).

**Static**
```go
package main

import (
	"github.com/kataras/iris"
)

func main() {
	api := iris.New()

	// first the subdomains.
	admin := api.Party("admin.")
	{
		// admin.mydomain.com
		admin.Get("/", func(c *iris.Context) {
			c.Write("INDEX FROM admin.mydomain.com")
		})
		// admin.mydomain.com/hey
		admin.Get("/hey", func(c *iris.Context) {
			c.Write("HEY FROM admin.mydomain.com/hey")
		})
		// admin.mydomain.com/hey2
		admin.Get("/hey2", func(c *iris.Context) {
			c.Write("HEY SECOND FROM admin.mydomain.com/hey")
		})
	}

	// mydomain.com/
	api.Get("/", func(c *iris.Context) {
		c.Write("INDEX FROM no-subdomain hey")
	})

	// mydomain.com/hey
	api.Get("/hey", func(c *iris.Context) {
		c.Write("HEY FROM no-subdomain hey")
	})

	api.Listen("mydomain.com:80")
}

```

**Dynamic/Wildcard **

```go

// Package main an example on how to catch dynamic subdomains - wildcard.
// On the first example (subdomains_1) we saw how to create routes for static subdomains, subdomains you know that you will have.
// Here we will see an example how to catch unknown subdomains, dynamic subdomains, like username.mydomain.com:8080.

package main

import "github.com/kataras/iris"

// register a dynamic-wildcard subdomain to your server machine(dns/...) first, check ./hosts if you use windows.
// run this file and try to redirect: http://username1.mydomain.com:8080/ , http://username2.mydomain.com:8080/ , http://username1.mydomain.com/something, http://username1.mydomain.com/something/sadsadsa

func main() {
	/* Keep note that you can use both of domains now (after 3.0.0-rc.1)
	   admin.mydomain.com,  and for other the Party(*.) but this is not this example's purpose

	admin := iris.Party("admin.")
	{
		// admin.mydomain.com
		admin.Get("/", func(c *iris.Context) {
			c.Write("INDEX FROM admin.mydomain.com")
		})
		// admin.mydomain.com/hey
		admin.Get("/hey", func(c *iris.Context) {
			c.Write("HEY FROM admin.mydomain.com/hey")
		})
		// admin.mydomain.com/hey2
		admin.Get("/hey2", func(c *iris.Context) {
			c.Write("HEY SECOND FROM admin.mydomain.com/hey")
		})
	}*/

	dynamicSubdomains := iris.Party("*.")
	{
		dynamicSubdomains.Get("/", dynamicSubdomainHandler)

		dynamicSubdomains.Get("/something", dynamicSubdomainHandler)

		dynamicSubdomains.Get("/something/:param1", dynamicSubdomainHandlerWithParam)
	}

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Write("Hello from mydomain.com path: %s", ctx.PathString())
	})

	iris.Get("/hello", func(ctx *iris.Context) {
		ctx.Write("Hello from mydomain.com path: %s", ctx.PathString())
	})

	iris.Listen("mydomain.com:8080")
}

func dynamicSubdomainHandler(ctx *iris.Context) {
	username := ctx.Subdomain()
	ctx.Write("Hello from dynamic subdomain path: %s, here you can handle the route for dynamic subdomains, handle the user: %s", ctx.PathString(), username)
	// if  http://username4.mydomain.com:8080/ prints:
	// Hello from dynamic subdomain path: /, here you can handle the route for dynamic subdomains, handle the user: username4
}

func dynamicSubdomainHandlerWithParam(ctx *iris.Context) {
	username := ctx.Subdomain()
	ctx.Write("Hello from dynamic subdomain path: %s, here you can handle the route for dynamic subdomains, handle the user: %s", ctx.PathString(), username)
	ctx.Write("THE PARAM1 is: %s", ctx.Param("param1"))
}


```

> You can still set unlimitted number of middleware/handlers to the dynamic subdomains also