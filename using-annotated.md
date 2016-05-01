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

```go
///file: main.go
//...cache the html files, if you the content of any html file changed, the templates are auto-reloading
iris.Config().Render.Directory = "./templates"
//...register the handler
iris.HandleAnnotated(&UserHandler{})
//...continue writing your wonderful API

```