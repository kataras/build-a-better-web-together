# Installation

Iris is a cross-platform software.

The only requirement is the [Go Programming Language](https://golang.org/dl/), version 1.13 and above.

```bash
$ go env -w GO111MODULE=on
```

## Install

```bash
$ go get github.com/kataras/iris/v12@master
```

Or edit your project's `go.mod` file.

```bash
module your_project_name

go 1.14

require (
    github.com/kataras/iris/v12 v12.1.9-0.20200618063647-c11725ab44d1
)
```

> `$ go build`

## Troubleshooting

If you get a network error during installation please make sure you set a valid [GOPROXY environment variable](https://github.com/golang/go/wiki/Modules#are-there-always-on-module-repositories-and-enterprise-proxies).

```bash
go env -w GOPROXY=https://goproxy.cn,https://gocenter.io,https://goproxy.io,direct
```


<!-- slide:break-100 -->
