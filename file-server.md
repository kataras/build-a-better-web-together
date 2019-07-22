# File Server

Serve static files from a specific directory \(system physical or embedded to the application\) is done by the `Party.HandleDir` method.

`HandleDir` registers a handler that serves HTTP requests with the contents of a file system \(physical or embedded\).

* First parameter  : the route path
* Second parameter : the system or the embedded directory that needs to be served
* Third parameter  : not required, the directory options, set fields is optional.

Returns the GET \*Route.

```go
HandleDir(requestPath, directory string, opts ...DirOptions) (getRoute *Route)
```

The `DirOptions` structure looks like this:

```go
type DirOptions struct {
    // Defaults to "/index.html", if request path is ending with **/*/$IndexName
    // then it redirects to **/*(/) which another handler is handling it,
    // that another handler, called index handler, is auto-registered by the framework
    // if end developer wasn't managed to handle it manually/by hand.
    IndexName string
    // Should files served under gzip compression?
    Gzip bool

    // List the files inside the current requested directory if `IndexName` not found.
    ShowList bool
    // If `ShowList` is true then this function will be used instead
    // of the default one to show the list of files of a current requested directory(dir).
    DirList func(ctx context.Context, dirName string, dir http.File) error

    // When embedded.
    Asset      func(name string) ([]byte, error)   
    AssetInfo  func(name string) (os.FileInfo, error)
    AssetNames func() []string

    // Optional validator that loops through each found requested resource.
    AssetValidator func(ctx context.Context, name string) bool
}
```

Let's say that you have an `./assets` folder near to your executable and you want the files to be served through `http://localhost:8080/static/**/*` route.

```go
app := iris.New()

app.HandleDir("/static", "./assets")

app.Run(iris.Addr(":8080"))
```

Now, if you want to embed the static files to be lived inside the executable build in order to not depend on a system directory you can use a tool like [go-bindata](https://github.com/go-bindata/go-bindata) to convert the files into `[]byte` inside your program. Let's take a quick tutorial on this and how Iris helps to serve those data.

Install go-bindata:

```bash
go get -u github.com/go-bindata/go-bindata/...
```

Navigate to your program directory, that the `./assets` subdirectory exists and execute:

```bash
$ go-bindata ./assets/...
```

The above creates a generated go file which contains three main functions: `Asset`, `AssetInfo` and `AssetNames`. Use them on the `DirOptions`:

```go
// [app := iris.New...]

app.HandleDir("/static", "./assets", iris.DirOptions {
    Asset: Asset,
    AssetInfo: AssetInfo,
    AssetNames: AssetNames,
    Gzip: false,
})
```

Build your app:

```bash
$ go build
```

The `HandleDir` supports all the standards, including content-range, for both physical and embedded directories.

However, if you just need a handler to work with, without register a route, you can use the `iris.FileServer` package-level function instead.

The `FileServer` function returns a Handler which serves files from a specific system, phyisical, directory or an embedded one.

* First parameter:  is the directory.
* Second parameter: optional parameter is any optional settings that the caller can use.

```go
iris.FileServer(directory string, options ...DirOptions)
```

**Usage**

```go
handler := iris.FileServer("./assets", iris.DirOptions {
    ShowList: true, Gzip: true, IndexName: "index.html",
})
```

Examples can be found at: [https://github.com/kataras/iris/tree/v11.2.0/\_examples/file-server](https://github.com/kataras/iris/tree/v11.2.0/_examples/file-server)

