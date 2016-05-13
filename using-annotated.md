# Using Annotated

Implements the Handler interface

```go
///file: userhandler.go
import "github.com/kataras/iris"

type UserHandler struct {
	iris.Handler `get:"/profile/user/:userId"`
}

func (u *UserHandler) Serve(c *iris.Context) {
	userId := c.Param("userId")
	c.Render("user.html", struct{ Message string }{Message: "Hello User with ID: " + userId})
}

```

```go
///file: main.go
iris.Config().Templates.Directory = "templates" // Default is already "templates"
//...register the handler
iris.HandleAnnotated(&UserHandler{})
//...continue writing your wonderful API

```