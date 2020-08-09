# Inputs

Here is a list of the built-in dependencies that can be used right away as input parameters:

| Type | Maps To |
| :--- | :--- |
| [*mvc.Application](https://pkg.go.dev/github.com/kataras/iris/v12/mvc?tab=doc#Application) | Current MVC Application |
| [iris.Context](https://pkg.go.dev/github.com/kataras/iris/v12/context?tab=doc#Context) | Current Iris Context |
| [*sessions.Session](https://pkg.go.dev/github.com/kataras/iris/v12/sessions?tab=doc#Session) | Current Iris Session |
| [context.Context](https://golang.org/pkg/context/#Context) | [ctx.Request().Context()](https://golang.org/pkg/net/http/#Request.Context) |
| [*http.Request](https://golang.org/pkg/net/http/#Request) | `ctx.Request()` |
| [http.ResponseWriter](https://golang.org/pkg/net/http/#ResponseWriter) | `ctx.ResponseWriter()` |
| [http.Header](https://golang.org/pkg/net/http/#Header) | `ctx.Request().Header` |
| [time.Time](https://golang.org/pkg/time/#Time) | `time.Now()` |
| [*golog.Logger](https://pkg.go.dev/github.com/kataras/golog) | Iris Logger |
| [net.IP](https://golang.org/pkg/net/#IP) | `net.ParseIP(ctx.RemoteAddr())` |
| `string`, |  |
| `int, int8, int16, int32, int64`, |  |
| `uint, uint8, uint16, uint32, uint64`, |  |
| `float, float32, float64`, |  |
| `bool`, |  |
| `slice` | [Path Parameter](https://github.com/kataras/iris/wiki/Routing-path-parameter-types) |
| Struct | [Request Body](https://github.com/kataras/iris/tree/master/_examples/request-body) of `JSON`, `XML`, `YAML`, `Form`, `URL Query`, `Protobuf`, `MsgPack` |

<!-- slide:break-80 -->
