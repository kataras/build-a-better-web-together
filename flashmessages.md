# Flash messages

**A flash message is used in order to keep a message in session through one or several requests of the same user**.   
By default, it is removed from the session after it has been displayed to the user.   
Flash messages are usually used in combination with HTTP redirections, because in this case there is no view, so messages can only be displayed in the request that follows redirection.

**A flash message has a name and a content (AKA key and value). It is an entry of a map**.   
The name is a string: often "notice", "success", or "error", but it can be anything. The content is usually a string. You can put HTML tags in your message if you display it raw. You can also set the message value to a number or an array: it will be serialized and kept in session like a string.

----


```go

// SetFlash sets a flash message, accepts 2 parameters the key(string) and the value(string)
// the value will be available on the NEXT request
SetFlash(key string, value string)

// GetFlash gets a flash message by it's key
// returns the value as string and an error
//
// if the cookie doesn't exists the string is empty and the error is filled.
// after the request's life the value is removed
GetFlash(key string) (value string, err error)

// GetFlashes returns all the flash messages which are available for this request 
GetFlashes() map[string]string
```

Example

```go

package main

import (
	"gopkg.in/kataras/iris.v5"
)

func main() {

	iris.Get("/set", func(c *iris.Context) {
		c.SetFlash("name", "iris")
		c.Write("Message set, is available for the next request")
	})

	iris.Get("/get", func(c *iris.Context) {
		name, err := c.GetFlash("name")
		if err != nil {
			c.Write(err.Error())
			return
		}
		c.Write("Hello %s", name)
	})

	iris.Get("/test", func(c *iris.Context) {

		name, err := c.GetFlash("name")
		if err != nil {
			c.Write(err.Error())
			return
		}

		c.Write("Ok you are comming from /set, the value of the name is %s", name)
		c.Write(", and again from the same context: %s", name)

	})

	iris.Listen(":8080")
}


```