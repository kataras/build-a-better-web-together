# Named Parameters

Named parameters are just custom paths to your routes, you can access them for each request using context's **c.Param("nameoftheparameter")**. Get all, as array (**{Key,Value}**) using **c.Params** property.

No limit on how long a path can be.

Usage:


```go
package main

import "github.com/kataras/iris"

func main() {
	// MATCH to /hello/anywordhere  (if PathCorrection:true match also /hello/anywordhere/)
	// NOT match to /hello or /hello/ or /hello/anywordhere/something
	iris.Get("/hello/:name", func(c *iris.Context) {
		name := c.Param("name")
		c.Write("Hello %s", name)
	})

	// MATCH to /profile/iris/friends/42  
    // (if PathCorrection:true matches also /profile/iris/friends/42/ ,otherwise not match)
	// NOT match to /profile/ , /profile/something ,
	// NOT match to /profile/something/friends,  /profile/something/friends ,
	// NOT match to /profile/anything/friends/42/something
	iris.Get("/profile/:fullname/friends/:friendId",
		func(c *iris.Context){
			name:= c.Param("fullname")
			//friendId := c.ParamInt("friendId")
			c.WriteHTML(iris.StatusOK,"<b> Hello </b>"+name)
		})

	iris.Listen(":8080")
}

```

### Match anything

```go
// Will match any request which url's preffix is "/anything/" and has content after that
iris.Get("/anything/*randomName", func(c *iris.Context) { } )
// Match: /anything/whateverhere/whateveragain , /anything/blablabla
// c.Param("randomName") will be /whateverhere/whateveragain, blablabla
// Not Match: /anything , /anything/ , /something
```
