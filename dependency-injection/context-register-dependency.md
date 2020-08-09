# Register Dependency from Context

When you want to create a middleware of `iris.Handler` form but you want to bind any input arguments of a potential (MVC) Controller's method or dependency-injection handlers.

The `ctx.RegisterDependency` is the method that allows you to build and add request-time dependencies.

```go
// RegisterDependency registers a struct or slice
// or pointer to struct dependency at request-time
// for the next handler in the chain. One value per type.
// Note that it's highly recommended to register
// your dependencies before server ran
// through Party.ConfigureContainer or mvc.Application.Register
// in sake of minimum performance cost.
RegisterDependency(v interface{})
// UnregisterDependency removes a dependency based on its type.
// Reports whether a dependency with that type was
// found and removed successfully.
UnregisterDependency(typ reflect.Type) bool
```

Let's start by creating a custom middleware that we can use in our hypothetical app.

```go
// Role struct value example.
type Role struct {
	Name string
}

const roleContextKey = "myapp.role"

// RoleMiddleware example of a custom middleware.
func RoleMiddleware(ctx iris.Context) {
	// [do it yourself: extract the role from the request...]
	if ctx.URLParam("name") != "kataras" {
		ctx.StopWithStatus(iris.StatusUnauthorized)
		return
	}
	//

	role := Role{Name: "admin"}

    // Share the role value to the next handler(s).
	ctx.Values().Set(roleContextKey, role)
```

> When you have access to the middleware itself:
Use the `RegisterDependency` to register
struct type values as dependencies at request-time for
any potential dependency injection-ed user handler.
This way the user of your middleware can get rid of
manually register a dependency for that `Role` type with calls of
`APIContainer.RegisterDependency` (and `mvc.Application.Register`).

```go
    ctx.RegisterDependency(role)
```

```go
	ctx.Next()
}
```

```go
// GetRole returns the role inside the context values,
// the `roleMiddleware` should be executed first.
func GetRole(ctx iris.Context) (Role, bool) {
	v := ctx.Values().Get(roleContextKey)
	if v != nil {
		if role, ok := v.(Role); ok {
			return role, true
		}
	}

	return Role{}, false
}
```

It's time to use our `RoleMiddleware`, in a common `iris.Handler` and a handler which accepts one or more dependencies. Create a **main.go** file and copy-paste the following code:

```go
package main

import "github.com/kataras/iris/v12"

func main() {
	app := iris.New()
	app.Use(RoleMiddleware)

	app.Get("/", commonHandler)
    c := app.ConfigureContainer()
```

> When you do NOT have access to the middleware code itself
then you can register a request dependency
which retrieves the value from the Context
and returns it, so handler/function's input arguments
with that `Role` type can be binded.

```go
    c.RegisterDependency(func(ctx iris.Context) Role {
        role, ok := GetRole(ctx)
        if !ok {
            // This codeblock will never be executed here
            // but you can stop executing a handler which depends on
            // that dependency with
            // `ctx.StopExecution/ctx.StopWithXXX` methods
            // or by returning a second output argument of `error` type.
            ctx.StopExecution()
            return Role{}
        }

        return role
    })
```

```go
	c.Get("/dep", handlerWithDependencies)

	// http://localhost:8080?name=kataras
	// http://localhost:8080/dep?name=kataras
	app.Listen(":8080")
}

func commonHandler(ctx iris.Context) {
	role, _ := GetRole(ctx)
	ctx.WriteString(role.Name)
}

func handlerWithDependencies(role Role) string {
	return role.Name
}
```

That's all.

<!-- slide:break-100 -->
