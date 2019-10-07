Iris is a cross-platform software.

The only requirement is the [Go Programming Language](https://golang.org/dl/), version 1.13 and above.

```sh
$ cd $YOUR_PROJECT_PATH
$ export GO111MODULE=on
```

## Install

```sh
$ go get github.com/kataras/iris@master
```

Or edit your project's `go.mod` file, add the following [pseudo-version](https://golang.org/cmd/go/#hdr-Pseudo_versions) and execute `$ go build`.

```sh
module your_project_name

go 1.13

require (
    github.com/kataras/iris v0.0.0-20191005193354-55afd07befa8
)
```

> `$ go build`

## How to update

Here is the go-get command to get the latest and greatest Iris version. Master branch is usually stable enough.

```bash
$ go get -u github.com/kataras/iris
```

## Troubleshooting

If you get a network error during installation please make sure you set a valid [GOPROXY environment variable](https://github.com/golang/go/wiki/Modules#are-there-always-on-module-repositories-and-enterprise-proxies) e.g. `GOPROXY=https://goproxy.io` or `GOPROXY=https://goproxy.cn`.

Continue by reading our [Getting Started](getting-started.md) tutorial.
