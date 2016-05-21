# Static files

Serve a static directory

```go

// StaticHandlerFunc returns a HandlerFunc to serve static system directory
// Accepts 5 parameters
//
// first is the systemPath (string)
// Path to the root directory to serve files from.
//
// second is the  stripSlashes (int) level
// * stripSlashes = 0, original path: "/foo/bar", result: "/foo/bar"
// * stripSlashes = 1, original path: "/foo/bar", result: "/bar"
// * stripSlashes = 2, original path: "/foo/bar", result: ""
//
// third is the compress (bool)
// Transparently compresses responses if set to true.
//
// The server tries minimizing CPU usage by caching compressed files.
// It adds FSCompressedFileSuffix suffix to the original file name and
// tries saving the resulting compressed file under the new file name.
// So it is advisable to give the server write access to Root
// and to all inner folders in order to minimze CPU usage when serving
// compressed responses.
//
// fourth is the generateIndexPages (bool)
// Index pages for directories without files matching IndexNames
// are automatically generated if set.
//
// Directory index generation may be quite slow for directories
// with many files (more than 1K), so it is discouraged enabling
// index pages' generation for such directories.
//
// fifth is the indexNames ([]string)
// List of index file names to try opening during directory access.
//
// For example:
//
//     * index.html
//     * index.htm
//     * my-super-index.xml
//
StaticHandlerFunc(systemPath string, stripSlashes int, compress bool,
                  generateIndexPages bool, indexNames []string) HandlerFunc 

// Static registers a route which serves a system directory
// this doesn't generates an index page which list all files
// no compression is used also, for these features look at StaticFS func
// accepts three parameters
// first parameter is the request url path (string)
// second parameter is the system directory (string)
// third parameter is the level (int) of stripSlashes
// * stripSlashes = 0, original path: "/foo/bar", result: "/foo/bar"
// * stripSlashes = 1, original path: "/foo/bar", result: "/bar"
// * stripSlashes = 2, original path: "/foo/bar", result: ""
Static(relative string, systemPath string, stripSlashes int)

// StaticFS registers a route which serves a system directory
// generates an index page which list all files
// uses compression which file cache, if you use this method it will generate compressed files also
// think this function as small fileserver with http
// accepts three parameters
// first parameter is the request url path (string)
// second parameter is the system directory (string)
// third parameter is the level (int) of stripSlashes
// * stripSlashes = 0, original path: "/foo/bar", result: "/foo/bar"
// * stripSlashes = 1, original path: "/foo/bar", result: "/bar"
// * stripSlashes = 2, original path: "/foo/bar", result: ""
StaticFS(relative string, systemPath string, stripSlashes int)

// StaticWeb same as Static but if index.html e
// xists and request uri is '/' then display the index.html's contents
// accepts three parameters
// first parameter is the request url path (string)
// second parameter is the system directory (string)
// third parameter is the level (int) of stripSlashes
// * stripSlashes = 0, original path: "/foo/bar", result: "/foo/bar"
// * stripSlashes = 1, original path: "/foo/bar", result: "/bar"
// * stripSlashes = 2, original path: "/foo/bar", result: ""
StaticWeb(relative string, systemPath string, stripSlashes int)

// StaticServe serves a directory as web resource
// it's the simpliest form of the Static* functions
// Almost same usage as StaticWeb
// accepts only one required parameter which is the systemPath 
// ( the same path will be used to register the GET&HEAD routes)
// if second parameter is empty, otherwise the requestPath is the second parameter
// it uses gzip compression (compression on each request, no file cache)
StaticServe(systemPath string, requestPath ...string)

```
```go

iris.Static("/public", "./static/assets/", 1)
//-> /public/assets/favicon.ico
```

```go
iris.StaticFS("/ftp", "./myfiles/public", 1)
```

```go
iris.StaticWeb("/","./my_static_html_website", 1)
```

### Manual static file serving
```go
// ServeFile serves a view file, to send a file
to the client you should use the SendFile(serverfilename,clientfilename)
// receives two parameters
// filename/path (string)
// gzipCompression (bool)
//
// You can define your own "Content-Type" header also, after this function call
func (ctx *Context) ServeFile(filename string, gzipCompression bool) error {
```
Serve static individual file

```go

iris.Get("/txt", func(ctx *iris.Context) {
	ctx.ServeFile("./myfolder/staticfile.txt", false)
}

```

For example if you want manual serve static individual files dynamically you can do something like that: 

```go
package main

import (
	"strings"
	"github.com/kataras/iris"
	"github.com/kataras/iris/utils"
)

func main() {

	iris.Get("/*file", func(ctx *iris.Context) {
	  	   requestpath := ctx.Param("file")

			path := strings.Replace(requestpath, "/", utils.PathSeperator, -1)

			if !utils.DirectoryExists(path) {
				ctx.NotFound()
				return
			}

			ctx.ServeFile(path, false) // make this true to use gzip compression
	}
}

iris.Listen(":8080")

```
The previous example is almost identical with

```go
StaticServe(systemPath string, requestPath ...string)
```
```go
func main() {
  iris.StaticServe("./mywebpage")
  // Serves all files inside this directory to the GET&HEAD route: 0.0.0.0:8080/mywebpage
  // using gzip compression ( no file cache, for file cache with zipped files use the StaticFS)
  iris.Listen(":8080")
}

```

```go
func main() {
  iris.StaticServe("./static/mywebpage","/webpage")
  // Serves all files inside filesystem path ./static/myfiles to the GET&HEAD route: 0.0.0.0:8080/webpage
  iris.Listen(":8080")
}

```