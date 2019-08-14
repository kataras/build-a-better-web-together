# Installing Iris

Iris is a cross-platform software.

The only requirement is the [Go Programming Language](https://golang.org/dl/), version 1.12 and above.

```bash
$ go get github.com/kataras/iris@v11.2.6
```

Or inside your `go.mod` file:

```bash
module your_project_name

go 1.12

require (
    github.com/kataras/iris v11.2.6
)
```

## How to update

Here is the go-get command to get the latest and greatest Iris version. Master branch is usually stable enough.

```bash
$ go get github.com/kataras/iris@master
```

Continue by reading our [Getting Started](getting-started.md) tutorial.
