# Sessions
If you notice a bug or issue [post it here](https://github.com/kataras/go-sessions).


- Cleans the temp memory when a session is idle, and re-allocates it to the temp memory when it's necessary. 
The most used sessions are optimized to be in the front of the memory's list.

- Supports any type of database, currently only [Redis](https://github.com/kataras/go-sessions/tree/master/sessiondb/redis) and [LevelDB](https://github.com/kataras/go-sessions/tree/master/sessiondb/leveldb).


**A session can be defined as a server-side storage of information that is desired to persist throughout the user's interaction with the web application**.

Instead of storing large and constantly changing data via cookies in the user's browser (i.e. CookieStore), 
**only a unique identifier is stored on the client side** called a "session id". 
This session id is passed to the web server on every request. 
The web application uses the session id as the key for retrieving the stored data from the database/memory. The session data is then available inside the iris.Context.

----



Here you see two different ways to use the sessions, we are using the first in this example. There are no performance differences.

## How to use

```go
package main

import	"github.com/kataras/iris"

func main() {

	// These are the optional fields to configurate sessions, 
	// using the station's Config field (iris.Config.Sessions)

	// Cookie string, the session's client cookie name, for example: "qsessionid"
	Cookie string
	// DecodeCookie set it to true to decode the cookie key with base64 URLEncoding
	// Defaults to false
	DecodeCookie bool

	// Expires the duration of which the cookie must expires (created_time.Add(Expires)).
	// If you want to delete the cookie when the browser closes, set it to -1 but in this case, the server side's session duration is up to GcDuration
	//
	// Default infinitive/unlimited life duration(0)
	Expires time.Duration

	// CookieLength the length of the sessionid's cookie's value, let it to 0 if you don't want to change it
	// Defaults to 32
	CookieLength int

	// GcDuration every how much duration(GcDuration) the memory should be clear for unused cookies (GcDuration)
	// for example: time.Duration(2)*time.Hour. it will check every 2 hours if cookie hasn't be used for 2 hours,
	// deletes it from backend memory until the user comes back, then the session continue to work as it was
	//
	// Default 2 hours
	GcDuration time.Duration

	// DisableSubdomainPersistence set it to true in order dissallow your q subdomains to have access to the session cookie
	// defaults to false
	DisableSubdomainPersistence bool

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
		// get a specific key as a string.
		// returns an empty string if the key was not found.
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
		// destroy/removes the entire session and cookie
		c.SessionDestroy()
		c.Log("You have to refresh the page to completely remove the session (on browsers), so the name should NOT be empty NOW, is it?\n ame: %s\n\nAlso check your cookies in your browser's cookies, should be no field for localhost/127.0.0.1 (or whatever you use)", c.Session().GetString("name"))
		c.Write("You have to refresh the page to completely remove the session (on browsers), so the name should NOT be empty NOW, is it?\nName: %s\n\nAlso check your cookies in your browser's cookies, should be no field for localhost/127.0.0.1 (or whatever you use)", c.Session().GetString("name"))
	})

	iris.Listen(":8080")
	//iris.ListenTLS("0.0.0.0:443", "mycert.cert", "mykey.key")
}


```

Example with **Redis session database**, which is located [here](https://github.com/kataras/go-sessions/tree/master/sessiondb/redis).

```go
package main

import (
	"github.com/kataras/go-sessions/sessiondb/redis"
	"github.com/kataras/go-sessions/sessiondb/redis/service"
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

		// set session values
		c.Session().Set("name", "iris")

		// test if set here
		c.Write("All ok session set to: %s", c.Session().GetString("name"))
	})

	iris.Get("/get", func(c *iris.Context) {
		// get a specific key as a string.
		// returns an empty string if the key was not found.
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
		// destroy/removes the entire session and cookie
		c.SessionDestroy()
		c.Log("You have to refresh the page to completely remove the session (on browsers), so the name should NOT be empty NOW, is it?\n ame: %s\n\nAlso check your cookies in your browser's cookies, should be no field for localhost/127.0.0.1 (or what ever you use)", c.Session().GetString("name"))
		c.Write("You have to refresh the page to completely remove the session (on browsers), so the name should NOT be empty NOW, is it?\nName: %s\n\nAlso check your cookies in your browser's cookies, should be no field for localhost/127.0.0.1 (or what ever you use)", c.Session().GetString("name"))
	})

	iris.Listen(":8080")
}

```
