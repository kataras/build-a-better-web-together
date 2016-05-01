# Party

Let's party with Iris web framework!

```go
func main() {

    //log everything middleware

    iris.UseFunc(func(c *iris.Context) {
		println("[Global log] the requested url path is: ", c.PathString())
		c.Next()
	})

    // manage all /users
    users := iris.Party("/users",func(c *iris.Context) {
			println("LOG [/users...] This is the middleware for: ", c.PathString())
			c.Next()
		}))
    {

		users.Post("/login", loginHandler)
        users.Get("/:userId", singleUserHandler)
        users.Delete("/:userId", userAccountRemoveUserHandler)
    }



    // Party inside an existing Party example:

    beta:= iris.Party("/beta")

    admin := beta.Party("/admin")
    {
		/// GET: /beta/admin/
		admin.Get("/", func(c *iris.Context){})
		/// POST: /beta/admin/signin
        admin.Post("/signin", func(c *iris.Context){})
		/// GET: /beta/admin/dashboard
        admin.Get("/dashboard", func(c *iris.Context){})
		/// PUT: /beta/admin/users/add
        admin.Put("/users/add", func(c *iris.Context){})
    }



    iris.Listen(":8080")
}
```