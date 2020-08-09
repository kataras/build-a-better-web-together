# Listing

By default Iris will not list files and sub-directories when client requests a path of a directory (e.g. `http://localhost:8080/files/folder`). To enable file listing you just set `DirOptions.ShowList` to `true`:

```go
options := iris.DirOptions{
    // [...]
    ShowList: true,
}

app.HandleDir("/files", iris.Dir("./uploads"), options)
```

By default listing is rendering a simple page of `<a href` html links, like the standard `net/http` package does. To change the default behavior use the `DirOptions.DirList` field, which accepts a function of form:

```go
type DirListFunc func(ctx iris.Context, dirOptions iris.DirOptions, dirName string, dir http.File) error
```

```go
options := iris.DirOptions{
    // [...]
    ShowList: true,
    DirList: func(ctx iris.Context, dirOptions iris.DirOptions, dirName string, dir http.File) error {
        // [...]
    },
}
```

The `DirListRich` is a function helper for a better look & feel. It's a `DirListFunc`.

```go
func DirListRich(opts ...DirListRichOptions) DirListFunc
```

The `DirListRich` function can optionally accept `DirListRichOptions`:

```go
type DirListRichOptions struct {
	// If not nil then this template's "dirlist" is used to render the listing page.
	Tmpl *template.Template // * "html/template"
	// If not empty then this view file is used to render the listing page.
	// The view should be registered with `Application.RegisterView`.
	// E.g. "dirlist.html"
	TmplName string
}
```

**Usage**

```go
options := iris.DirOptions{
    // [...]
    ShowList: true,
    DirList: iris.DirListRich(),
}
```

## Example

Here's a full example that users can upload files and server can list them. It contains a customized listing page and delete file feature with basic authentication.

```text
│   main.go # Iris web server.
└───views
│      upload.html # used to render upload form.
│      dirlist.html # used to render listing page.
└───uploads
│      # uploaded files will be stored here.
```

Create a **main.go** file and copy-paste the following contents:

```go
package main

import (
	"crypto/md5"
	"fmt"
	"io"
	"mime/multipart"
	"os"
	"path"
	"strconv"
	"strings"
	"time"

	"github.com/kataras/iris/v12"
	"github.com/kataras/iris/v12/middleware/basicauth"
)

func init() {
	os.Mkdir("./uploads", 0700)
}

const (
	maxSize   = 1 * iris.GB
	uploadDir = "./uploads"
)

func main() {
	app := iris.New()

	view := iris.HTML("./views", ".html")
	view.AddFunc("formatBytes", func(b int64) string {
		const unit = 1000
		if b < unit {
			return fmt.Sprintf("%d B", b)
		}
		div, exp := int64(unit), 0
		for n := b / unit; n >= unit; n /= unit {
			div *= unit
			exp++
		}
		return fmt.Sprintf("%.1f %cB",
			float64(b)/float64(div), "kMGTPE"[exp])
	})
	app.RegisterView(view)

	// Serve assets (e.g. javascript, css).
	// app.HandleDir("/public", "./public")

	app.Get("/", index)

	app.Get("/upload", uploadView)
	app.Post("/upload", upload)

	filesRouter := app.Party("/files")
	{
		filesRouter.HandleDir("/", iris.Dir(uploadDir), iris.DirOptions{
			Compress: true,
			ShowList: true,
			DirList: iris.DirListRich(iris.DirListRichOptions{
				// Optionally, use a custom template for listing:
				// Tmpl: dirListRichTemplate,
				TmplName: "dirlist.html",
			}),
		})

		auth := basicauth.New(basicauth.Config{
			Users: map[string]string{
				"myusername": "mypassword",
			},
		})

		filesRouter.Delete("/{file:path}", auth, deleteFile)
	}

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

func upload(ctx iris.Context) {
	ctx.SetMaxRequestBodySize(maxSize)

	_, err := ctx.UploadFormFiles(uploadDir, beforeSave)
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

func deleteFile(ctx iris.Context) {
	// It does not contain the system path,
	// as we are not exposing it to the user.
	fileName := ctx.Params().Get("file")

	filePath := path.Join(uploadDir, fileName)

	if err := os.RemoveAll(filePath); err != nil {
		ctx.StopWithError(iris.StatusInternalServerError, err)
		return
	}

	ctx.Redirect("/files")
}
```

The **views/upload.html** should look like that:

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

        <input type="button" value="Upload using JS" onclick="uploadFiles()" />
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

And finally the **customized listing page**. Copy-paste the following code to the **views/dirlist.html** file:

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{.Title}}</title>
    <style>
        a {
            padding: 8px 8px;
            text-decoration: none;
            cursor: pointer;
            color: #10a2ff;
        }

        table {
            position: absolute;
            top: 0;
            bottom: 0;
            left: 0;
            right: 0;
            height: 100%;
            width: 100%;
            border-collapse: collapse;
            border-spacing: 0;
            empty-cells: show;
            border: 1px solid #cbcbcb;
        }

        table caption {
            color: #000;
            font: italic 85%/1 arial, sans-serif;
            padding: 1em 0;
            text-align: center;
        }

        table td,
        table th {
            border-left: 1px solid #cbcbcb;
            border-width: 0 0 0 1px;
            font-size: inherit;
            margin: 0;
            overflow: visible;
            padding: 0.5em 1em;
        }

        table thead {
            background-color: #10a2ff;
            color: #fff;
            text-align: left;
            vertical-align: bottom;
        }

        table td {
            background-color: transparent;
        }

        .table-odd td {
            background-color: #f2f2f2;
        }

        .table-bordered td {
            border-bottom: 1px solid #cbcbcb;
        }

        .table-bordered tbody>tr:last-child>td {
            border-bottom-width: 0;
        }
    </style>
</head>

<body>
    <table class="table-bordered table-odd">
        <thead>
            <tr>
                <th>#</th>
                <th>Name</th>
                <th>Size</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
            {{ range $idx, $file := .Files }}
            <tr>
                <td>{{ $idx }}</td>
                <td><a href="{{ $file.Path }}" title="{{ $file.ModTime }}">{{ $file.Name }}</a></td>
                {{ if $file.Info.IsDir }}
                <td>Dir</td>
                {{ else }}
                <td>{{ formatBytes $file.Info.Size }}</td>
                {{ end }}

                <td><input type="button" style="background-color:transparent;border:0px;cursor:pointer;" value="❌"
                        onclick="deleteFile({{ $file.RelPath }})" /></td>
            </tr>
            {{ end }}
        </tbody>
    </table>
    <script type="text/javascript">
        function deleteFile(filename) {
            if (!confirm("Are you sure you want to delete " + filename + " ?")) {
                return;
            }

            fetch('/files/' + filename,
                {
                    method: "DELETE",
                    // If you don't want server to prompt for username/password:
                    // credentials:"include",
                    headers: {
                        // "Authorization": "Basic " + btoa("myusername:mypassword")
                        "X-Requested-With": "XMLHttpRequest",
                    },
                }).
                then(data => location.reload()).
                catch(e => alert(e));
        }
    </script>
</body>

</html>
```
