# Hi
The name of this framework came from **Greek mythology**, **Iris** was the name of the Greek goddess of the **rainbow**.

```go
package main

import "github.com/kataras/iris"

func main() {
	iris.Get("/hello", func(c *iris.Context) {
		c.Write("Hi %s", "iris")
	})
	iris.Listen(":8080") // or err := iris.ListenWithErr(":8080")
}

```
