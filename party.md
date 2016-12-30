# Party

Let's party with Iris web framework!

```go
package main

import "gopkg.in/kataras/iris.v5"

func main() {
    admin := iris.Party("/admin", 
	    func(ctx *iris.Context){
     	   ctx.Write("Middleware for all party's routes!")
		   ctx.Next();
		})
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

    iris.Listen(":8080")
}
```

