# Flash messages

**A flash message is used in order to keep a message in session through one or several requests of the same user**. By default, it is removed from session after it has been displayed to the user. Flash messages are usually used in combination with HTTP redirections, because in this case there is no view, so messages can only be displayed in the request that follows redirection.

**A flash message has a name and a content (AKA key and value). It is an entry of a map**. The name is a string: often "notice", "success", or "error", but it can be anything. The content is usually a string. You can put HTML tags in your message if you display it raw. You can also set the message value to a number or an array: it will be serialized and kept in session like a string.

```go
// GetFlash get a flash message by it's key 
// after this action the messages is removed
// returns string
// if the cookie doesn't exists the string is empty
GetFlash(key string) string

// GetFlashBytes get a flash message by it's key 
// after this action the messages is removed
// returns []byte 
// and an error if the cookie doesn't exists or decode fails
GetFlashBytes(key string) (value []byte, err error) 

// SetFlash sets a flash message
// accepts 2 parameters the key(string) and the value(string)
SetFlash(key string, value string)

// SetFlash sets a flash message
// accepts 2 parameters the key(string) and the value([]byte)
SetFlashBytes(key string, value []byte) 
```

Example

```go

package main

import (
	"github.com/kataras/iris"
)

func main() {

	iris.Get("/set", func(c *iris.Context) {
		c.SetFlash("name", "iris")
	})

	iris.Get("/get", func(c *iris.Context) {
		c.Write("Hello %s", c.GetFlash("name"))
        // the flash message is being deleted after this request done,
        // so you can call the c.GetFlash("name") 
        // many times without problem
	})

	iris.Get("/test", func(c *iris.Context) {

		name := c.GetFlash("name")
		if name == "" {
			c.Write("Ok you are comming from /get")
		} else {
			c.Write("Ok you are comming from /set: %s", name)
		}
	})

	iris.Listen(":8080")
}

```