# HTTP Method Override

The use of specific custom HTTP headers such as X-HTTP methods override can be very handy while developing and promoting a REST API. When deploying REST API based web services, you may encounter access limitations on both the **server** and **client** sides.

Some Firewalls do not support PUT, DELETE or PATCH requests.

The [Method Override](https://github.com/kataras/iris/tree/master/middleware/methodoverride) wrapper lets you use HTTP verbs such as PUT or DELETE in places where the client doesn't support it.

## Server

```go
package main

import (
    "github.com/kataras/iris/v12"
    "github.com/kataras/iris/v12/middleware/methodoverride"
)

func main() {
    app := iris.New() 

    mo := methodoverride.New( 
        // Defaults to nil. 
        // 
        methodoverride.SaveOriginalMethod("_originalMethod"), 
        // Default values. 
        // 
        // methodoverride.Methods(http.MethodPost), 
        // methodoverride.Headers("X-HTTP-Method",
        //                        "X-HTTP-Method-Override",
        //                        "X-Method-Override"), 
        // methodoverride.FormField("_method"), 
        // methodoverride.Query("_method"), 
    ) 
    // Register it with `WrapRouter`. 
    app.WrapRouter(mo)

    app.Post("/path", func(ctx iris.Context) {
        ctx.WriteString("post response")
    })

    app.Delete("/path", func(ctx iris.Context) {
        ctx.WriteString("delete response")
    })

    // [...app.Run]
}
```

## Client

```javascript
fetch("/path", {
    method: 'POST',
    headers: {
      "X-HTTP-Method": "DELETE"
    },
  })
  .then((resp)=>{
      // response body will be "delete response". 
 })).catch((err)=> { console.error(err) })
```

