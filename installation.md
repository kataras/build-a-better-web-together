Iris is a cross-platform software.

The only requirement is the [Go Programming Language](https://golang.org/dl/), version 1.13 and above.

```sh
$ cd $YOUR_PROJECT_PATH
$ export GO111MODULE=on
```

## Install

```sh
$ go get github.com/kataras/iris/v12@latest
```

Or edit your project's `go.mod` file.

```sh
module your_project_name

go 1.13

require (
    github.com/kataras/iris/v12 v12.1.0
)
```

> `$ go build`

<!-- ## How to update

Here is the go-get command to get the latest and greatest Iris version. Master branch is usually stable enough.

```bash
$ go get -u github.com/kataras/iris/v12@latest
``` -->

## Troubleshooting

If you get a network error during installation please make sure you set a valid [GOPROXY environment variable](https://github.com/golang/go/wiki/Modules#are-there-always-on-module-repositories-and-enterprise-proxies) e.g. `GOPROXY=https://goproxy.io` or `GOPROXY=https://goproxy.cn`.

Continue by reading our [[Getting Started]] tutorial.
