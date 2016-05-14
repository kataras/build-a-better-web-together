# Hi


```go
package main

import "github.com/kataras/iris"

func main() {
	iris.Get("/hello", func(ctx *iris.Context) {
		ctx.Write("Hi %s", "iris")
	})
	iris.Listen(":8080")
    //err := iris.ListenWithErr(":8080")
}

```

The same
```go
package main

import "github.com/kataras/iris"

func main() {
    api := iris.New()
	api.Get("/hello", hi)
	api.Listen(":8080")
}

func hi(ctx *iris.Context){
   ctx.Write("Hi %s", "iris")
}

```

Rich Hi with `html/template`

```html
<!-- templates/hi.html -->
<html><head> <title> Hi Iris [THE TITLE] </title> </head>
  <body>
    <h1> Hi {{.Name}}
  </body>
</html>


```

```go


```