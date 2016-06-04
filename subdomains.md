# Subdomains


Subdomains are splitted to two categories, first is the static subdomain and second is the dynamic subdomain.

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
	// first the subdomains.
	admin := iris.Party("admin.127.0.0.1")
	{
		//this will only success on admin.127.0.0.1/hey
		admin.Get("/", func(c *iris.Context) {
			c.Write("Welcome to admin.127.0.0.1/")
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

**Dynamic/Wildcard **

```go
// Package main an example on how to catch dynamic subdomains - wildcard.
// On the first example (subdomains_1) we saw how to create routes for static subdomains, subdomains you know that you will have.
// Here we will see an example how to catch unknown subdomains, 
// dynamic subdomains, like username.mysite.com.
package main

import "github.com/kataras/iris"

// register a dynamic-wildcard subdomain to your server machine(dns/...) first.
// run this file and try to redirect: http://admin.yourhost.com:8080/ ,
// http://admin.yourhost.com:8080/something, http://admin.yourhost.com:8080/something/sadsadsa

func main() {

	dynamicSubdomains := iris.Party("*.")
	{
		dynamicSubdomains.Get("/", dynamicSubdomainHandler)

		dynamicSubdomains.Get("/something", dynamicSubdomainHandler)

		dynamicSubdomains.Get("/something/:param1", dynamicSubdomainHandlerWithParam)
	}
    
	// without party:
	// iris.Wildcard("GET", "/", dynamicSubdomainHandler)
	// iris.Wildcard("GET", "/something", dynamicSubdomainHandler)
	// iris.Wildcard("GET", "/something/:param1", dynamicSubdomainHandlerWithParam)

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Write("Hello from localhost path: %s", ctx.PathString())
	})

	iris.Get("/hello", func(ctx *iris.Context) {
		ctx.Write("Hello from localhost path: %s", ctx.PathString())
	})

	iris.Listen("127.0.0.1:8080")
}

func dynamicSubdomainHandler(ctx *iris.Context) {
	username := ctx.Subdomain()
	ctx.Write("Hello from dynamic subdomain path: %s, here you can handle the route for dynamic subdomains, handle the user: %s", ctx.PathString(), username)
	// if  http://admin.yourhost.com:8080/ prints:
	// Hello from dynamic subdomain path: /, here you can handle the route for dynamic subdomains, handle the user: admin
}

func dynamicSubdomainHandlerWithParam(ctx *iris.Context) {
	username := ctx.Subdomain()
	ctx.Write("Hello from dynamic subdomain path: %s, here you can handle the route for dynamic subdomains, handle the user: %s", ctx.PathString(), username)
	ctx.Write("THE PARAM1 is: %s", ctx.Param("param1"))
}


```

> You can still set unlimitted number of middleware/handlers to the dynamic subdomains also