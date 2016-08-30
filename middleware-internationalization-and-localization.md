# Internationalization and Localization

[This is a middleware](https://github.com/iris-contrib/middleware/tree/master/i18n)

## Tutorial

Create folder named 'locales':
```
// Files: 

./locales/locale_en-US.ini 
./locales/locale_el-US.ini 
```
Contents on locale_en-US:
``` 
hi = hello, %s
``` 
Contents on locale_el-GR:
``` 
hi = Γειά, %s
``` 

```go

package main

import (
	"fmt"
	"github.com/kataras/iris"
	"github.com/iris-contrib/middleware/i18n"
)

func main() {

	iris.Use(i18n.New(i18n.Config{Default: "en-US",
		Languages: map[string]string{
			"en-US": "./locales/locale_en-US.ini",
			"el-GR": "./locales/locale_el-GR.ini",
			"zh-CN": "./locales/locale_zh-CN.ini"}}))	
	
	iris.Get("/", func(ctx *iris.Context) {
		hi := ctx.GetFmt("translate")("hi", "maki") // hi is the key, 'maki' is the %s, the second parameter is optional
		language := ctx.Get("language") // language is the language key, example 'en-US'

		ctx.Write("From the language %s translated output: %s", language, hi)
	})

	iris.Listen(":8080")

}

```