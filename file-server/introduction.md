# Introduction

Serve static files from a specific directory (system physical or embedded to the application) is done by the `Party.HandleDir` method.

`HandleDir` registers a handler that serves HTTP requests with the contents of a file system (physical or embedded).

* First parameter  : the route path
* Second parameter : the file system that needs to be served
* Third parameter  : optional, the directory options.

```go
HandleDir(requestPath string, fs http.FileSystem, opts ...DirOptions) ( []*Route)
```

The `DirOptions` structure looks like this:

```go
type DirOptions struct {
	// Defaults to "/index.html", if request path is ending with **/*/$IndexName
	// then it redirects to **/*(/) which another handler is handling it,
	// that another handler, called index handler, is auto-registered by the framework
	// if end developer does not managed to handle it by hand.
	IndexName string
	// PushTargets filenames (map's value) to
	// be served without additional client's requests (HTTP/2 Push)
	// when a specific request path (map's key WITHOUT prefix)
	// is requested and it's not a directory (it's an `IndexFile`).
	//
	// Example:
	// 	"/": {
	// 		"favicon.ico",
	// 		"js/main.js",
	// 		"css/main.css",
	// 	}
	PushTargets map[string][]string
	// PushTargetsRegexp like `PushTargets` but accepts regexp which
	// is compared against all files under a directory (recursively).
	// The `IndexName` should be set.
	//
	// Example:
	// "/": regexp.MustCompile("((.*).js|(.*).css|(.*).ico)$")
	// See `iris.MatchCommonAssets` too.
	PushTargetsRegexp map[string]*regexp.Regexp

	// Cache to enable in-memory cache and pre-compress files.
	Cache DirCacheOptions
	// When files should served under compression.
	Compress bool

	// List the files inside the current requested directory if `IndexName` not found.
	ShowList bool
	// If `ShowList` is true then this function will be used instead
	// of the default one to show the list of files of a current requested directory(dir).
	// See `DirListRich` package-level function too.
	DirList DirListFunc

	// Files downloaded and saved locally.
	Attachments Attachments

	// Optional validator that loops through each requested resource.
	AssetValidator func(ctx *context.Context, name string) bool
}
```

## Quick Start

Let's say that you have an `./assets` folder near to your executable and you want the files to be served through `http://localhost:8080/static/**/*` route.

```go
app := iris.New()

app.HandleDir("/static", iris.Dir("./assets"))

app.Listen(":8080")
```

Now, if you want to embed the static files to be lived inside the executable build in order to not depend on a system directory you can use a tool like [go-bindata](https://github.com/go-bindata/go-bindata) to convert the files into `[]byte` inside your program. Let's take a quick tutorial on this and how Iris helps to serve those data.

Install go-bindata:

```bash
$ go get -u github.com/go-bindata/go-bindata/v3/go-bindata
```

Navigate to your program directory, that the `./assets` subdirectory exists and execute:

```bash
$ go-bindata -fs -prefix "assets" ./assets/...
```

The above creates a generated go file which contains a `AssetFile()` functions that returns a compatible `http.FileSystem` you can give to Iris to serve the files.

```go
// [app := iris.New...]

app.HandleDir("/static", AssetFile())
```

Run your app:

```bash
$ go run . 
```

The `HandleDir` supports all the standards, including content-range, for both physical, embedded and cached directories.

However, if you just need a handler to work with, without register a route, you can use the `iris.FileServer` package-level function instead.

The `FileServer` function returns a Handler which serves files from a specific system directory, an embedded one or  a memory-cached one.

```go
iris.FileServer(fs http.FileSystem, options DirOptions)
```

**Usage**

```go
handler := iris.FileServer(iris.Dir("./assets"), iris.DirOptions {
    ShowList: true, Compress: true, IndexName: "index.html",
})
```

## Example

Let's create an application that users can upload one or more files and list them.

Copy-paste the contents of **main.go**:

```go
package main

import (
	"crypto/md5"
	"fmt"
	"io"
	"mime/multipart"
	"os"
	"strconv"
	"strings"
	"time"

	"github.com/kataras/iris/v12"
)

func init() {
	os.Mkdir("./uploads", 0700)
}

func main() {
	app := iris.New()
	app.RegisterView(iris.HTML("./views", ".html"))
	// Serve assets (e.g. javascript, css).
	app.HandleDir("/public", iris.Dir("./public"))

	app.Get("/", index)

	app.Get("/upload", uploadView)
	app.Post("/upload", upload)

	app.HandleDir("/files", iris.Dir("./uploads"), iris.DirOptions{
		Compress: true,
		ShowList: true,
	})

	app.Listen(":8080")
}

func index(ctx iris.Context) {
	ctx.Redirect("/upload")
}

func uploadView(ctx iris.Context) {
	now := time.Now().Unix()
	h := md5.New()
	io.WriteString(h, strconv.FormatInt(now, 10))
	token := fmt.Sprintf("%x", h.Sum(nil))

	ctx.View("upload.html", token)
}

const maxSize = 10 * iris.MB

func upload(ctx iris.Context) {
	ctx.SetMaxRequestBodySize(maxSize)

	_, err := ctx.UploadFormFiles("./uploads", beforeSave)
	if err != nil {
		ctx.StopWithError(iris.StatusPayloadTooRage, err)
		return
	}

	ctx.Redirect("/files")
}

func beforeSave(ctx iris.Context, file *multipart.FileHeader) {
	ip := ctx.RemoteAddr()
	ip = strings.ReplaceAll(ip, ".", "_")
	ip = strings.ReplaceAll(ip, ":", "_")

	file.Filename = ip + "-" + file.Filename
}
```

The **./views/upload.html** is a simple html form, looks like that:

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Upload Files</title>
</head>

<body>
    <form enctype="multipart/form-data" action="/upload" method="POST">
        <input type="file" id="upload_files" name="upload_files" multiple />
        <input type="hidden" name="token" value="{{.}}" />

        <input type="button" value="Upload Using JS" onclick="uploadFiles()" />
        <input type="submit" value="Upload by submiting the form" />
    </form>

    <script type="text/javascript">
        function uploadFiles() {
            let files = document.getElementById("upload_files").files;
            let formData = new FormData();
            for (var i = 0; i < files.length; i++) {
                formData.append("files[]", files[i]);
            }

            fetch('/upload',
                {
                    method: "POST",
                    body: formData
                }).
                then(data => window.location = "/files").
                catch(e => window.alert("upload failed: file too large"));
        }
    </script>
</body>

</html>
```
<!-- slide:break-80 -->
