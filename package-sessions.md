# Sessions
If you notice a bug or issue [post it here](https://github.com/kataras/iris/issues)


- Cleans the temp memory when a sessions is iddle, and re-allocate it , fast, to the temp memory when it's necessary. Also most used/regular sessions are going front in the memory's list.

- Supports any type of database, currently only [redis](https://github.com/iris-contrib/sessiondb/).


**A session can be defined as a server-side storage of information that is desired to persist throughout the user's interaction with the web site** or web application.

Instead of storing large and constantly changing information via cookies in the user's browser, **only a unique identifier is stored on the client side** (called a "session id"). This session id is passed to the web server every time the browser makes an HTTP request (ie a page link or AJAX request). The web application pairs this session id with it's internal database/memory and retrieves the stored variables for use by the requested page.

----



You will see two different ways to use the sessions, I'm using the first. No performance differences.

## How to use

```go
package main

import	"github.com/kataras/iris"

func main() {
	/*  These are the optionally fields to configurate the sessions, using the station's Config field (iris.Config.Sessions)

	// Cookie string, the session's client cookie name, for example: "irissessionid"
	Cookie string
	// DecodeCookie set it to true to decode the cookie key with base64 URLEncoding
	// Defaults to false
	DecodeCookie bool
	// Expires the duration of which the cookie must expires (created_time.Add(Expires)).
	// Default infinitive/unlimited life duration(0)
	Expires time.Duration
	// GcDuration every how much duration(GcDuration) the memory should be clear for unused cookies (GcDuration)
	// for example: time.Duration(2)*time.Hour. it will check every 2 hours if cookie hasn't be used for 2 hours,
	// deletes it from backend memory until the user comes back, then the session continue to work as it was
	//
	// Default 2 hours
	GcDuration time.Duration
	// DisableSubdomainPersistence set it to true in order dissallow your iris subdomains to have access to the session cookie
	// defaults to false
	DisableSubdomainPersistence bool
	*/
	iris.Get("/", func(c *iris.Context) {
		c.Write("You should navigate to the /set, /get, /delete, /clear,/destroy instead")
	})
	iris.Get("/set", func(c *iris.Context) {

		//set session values
		c.Session().Set("name", "iris")

		//test if setted here
		c.Write("All ok session setted to: %s", c.Session().GetString("name"))
	})

	iris.Get("/get", func(c *iris.Context) {
		// get a specific key, as string, if no found returns just an empty string
		name := c.Session().GetString("name")

		c.Write("The name on the /set was: %s", name)
	})

	iris.Get("/delete", func(c *iris.Context) {
		// delete a specific key
		c.Session().Delete("name")
	})

	iris.Get("/clear", func(c *iris.Context) {
		// removes all entries
		c.Session().Clear()
	})

	iris.Get("/destroy", func(c *iris.Context) {
		//destroy, removes the entire session and cookie
		c.SessionDestroy()
		c.Log("You have to refresh the page to completely remove the session (on browsers), so the name should NOT be empty NOW, is it?\n ame: %s\n\nAlso check your cookies in your browser's cookies, should be no field for localhost/127.0.0.1 (or what ever you use)", c.Session().GetString("name"))
		c.Write("You have to refresh the page to completely remove the session (on browsers), so the name should NOT be empty NOW, is it?\nName: %s\n\nAlso check your cookies in your browser's cookies, should be no field for localhost/127.0.0.1 (or what ever you use)", c.Session().GetString("name"))
	})

	iris.Listen(":8080")
	//iris.ListenTLS("0.0.0.0:443", "mycert.cert", "mykey.key")
}


```

Example with **redis session database**, which located [here](https://github.com/iris-contrib/sessiondb/tree/master/redis)

```go
package main

import (
	"github.com/iris-contrib/sessiondb/redis"
	"github.com/iris-contrib/sessiondb/redis/service"
	"github.com/kataras/iris"
)

func main() {
	db := redis.New(service.Config{Network: service.DefaultRedisNetwork,
		Addr:          service.DefaultRedisAddr,
		Password:      "",
		Database:      "",
		MaxIdle:       0,
		MaxActive:     0,
		IdleTimeout:   service.DefaultRedisIdleTimeout,
		Prefix:        "",
		MaxAgeSeconds: service.DefaultRedisMaxAgeSeconds}) // optionally configure the bridge between your redis server

	iris.UseSessionDB(db)

	iris.Get("/set", func(c *iris.Context) {

		//set session values
		c.Session().Set("name", "iris")

		//test if setted here
		c.Write("All ok session setted to: %s", c.Session().GetString("name"))
	})

	iris.Get("/get", func(c *iris.Context) {
		// get a specific key, as string, if no found returns just an empty string
		name := c.Session().GetString("name")

		c.Write("The name on the /set was: %s", name)
	})

	iris.Get("/delete", func(c *iris.Context) {
		// delete a specific key
		c.Session().Delete("name")
	})

	iris.Get("/clear", func(c *iris.Context) {
		// removes all entries
		c.Session().Clear()
	})

	iris.Get("/destroy", func(c *iris.Context) {
		//destroy, removes the entire session and cookie
		c.SessionDestroy()
		c.Log("You have to refresh the page to completely remove the session (on browsers), so the name should NOT be empty NOW, is it?\n ame: %s\n\nAlso check your cookies in your browser's cookies, should be no field for localhost/127.0.0.1 (or what ever you use)", c.Session().GetString("name"))
		c.Write("You have to refresh the page to completely remove the session (on browsers), so the name should NOT be empty NOW, is it?\nName: %s\n\nAlso check your cookies in your browser's cookies, should be no field for localhost/127.0.0.1 (or what ever you use)", c.Session().GetString("name"))

	})

	iris.Listen(":8080")
}

```
