# Party

Let's party with Iris web framework!

```go
package main

import "github.com/kataras/iris"

func main() {
    admin := iris.Party("/admin", func(ctx *iris.Context){ ctx.Write("Middleware for all party's routes!" })
    {
        // add a silly middleware
        admin.UseFunc(func(c *iris.Context) {
            //your authentication logic here...
            println("from ", c.PathString())
            authorized := true
            if authorized {
                c.Next()
            } else {
                c.Text(401, c.PathString()+" is not authorized for you")
            }

        })
        admin.Get("/", func(c *iris.Context) {
            c.Write("from /admin/ or /admin if you pathcorrection on")
        })
        admin.Get("/dashboard", func(c *iris.Context) {
            c.Write("/admin/dashboard")
        })
        admin.Delete("/delete/:userId", func(c *iris.Context) {
            c.Write("admin/delete/%s", c.Param("userId"))
        })
    }


    beta := admin.Party("/beta")
    beta.Get("/hey", func(c *iris.Context) { c.Write("hey from /admin/beta/hey") })

    //for subdomains goto: ../subdomains_1/main.go

    iris.Listen(":8080")

}
```

