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
	c.Render("user", struct{ Message string }{Message: "Hello User with ID: " + userId})
}

```