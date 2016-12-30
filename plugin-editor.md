# Editor

[This is a plugin](https://github.com/iris-contrib/plugin/tree/5.0.0/editor).

Editor Plugin is just a bridge between Iris and [alm-tools](http://alm.tools).


[alm-tools](http://alm.tools) is a typescript online IDE/Editor, made by [@basarat](https://twitter.com/basarat) 
one of the top contributors of the [Typescript](http://www.typescriptlang.org) language.

Iris gives you the opportunity to edit your client-side using the alm-tools editor, via the editor plugin.


This plugin starts it's own server. If Iris server is using TLS then the editor will use the same key and cert.

## How to use

```go
package main

import (
	"gopkg.in/kataras/iris.v5"
	"github.com/iris-contrib/plugin/editor"
)

func main(){
	e := editor.New() 
	// editor.Config{ Username: "admin", Password: "admin!123", Port: 4444, WorkingDir: "/public/scripts"}

	iris.Plugins.Add(e)

	iris.Get("/", func (ctx *iris.Context){})

	iris.Listen(":8080")
}


```

**Note for username, password**: The Authorization specifies the authentication mechanism (in this case Basic) followed by the username and password.
Although the string aHR0cHdhdGNoOmY= may look encrypted it is simply a base64 encoded version of username:password.
Would be readable to anyone who could intercept the HTTP request (if TLS is not used). [Read more here](https://www.httpwatch.com/httpgallery/authentication).

> The editor can't work if the directory doesn't contain a [tsconfig.json](http://www.typescriptlang.org/docs/handbook/tsconfig.json.html).

> If you are using the [typescript plugin](https://github.com/iris-contrib/plugin/tree/5.0.0/typescript) you don't have to call the .Dir(...)


