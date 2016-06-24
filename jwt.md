## JSON Web Tokens


This is a [middleware](https://github.com/iris-contrib/middleware/jwt).

## What is it?

[JWT.io](https://jwt.io) has a great [introduction](https://jwt.io/introduction/) to JSON Web Tokens.

In short, it's a signed JSON object that does something useful (for example, authentication). It's commonly used for Bearer tokens in Oauth 2. A token is made of three parts, separated by .'s. The first two parts are JSON objects, that have been base64url encoded. The last part is the signature, encoded the same way.

The first part is called the header. It contains the necessary information for verifying the last part, the signature. For example, which encryption method was used for signing and what key was used.

The part in the middle is the interesting bit. It's called the Claims and contains the actual stuff you care about. Refer to the RFC for information about reserved keys and the proper way to add your own.


## Example
```go
package main

import (
	"github.com/dgrijalva/jwt-go"
	jwtmiddleware "github.com/iris-contrib/middleware/jwt"
	"github.com/kataras/iris"
)

func main() {

	myJwtMiddleware := jwtmiddleware.New(jwtmiddleware.Config{
		ValidationKeyGetter: func(token *jwt.Token) (interface{}, error) {
			return []byte("My Secret"), nil
		},
		SigningMethod: jwt.SigningMethodHS256,
	})

	iris.Get("/ping", PingHandler)
	iris.Get("/secured/ping", myJwtMiddleware.Serve, SecuredPingHandler)
	iris.Listen(":8080")
}

type Response struct {
	Text string `json:"text"`
}

func PingHandler(ctx *iris.Context) {
	response := Response{"All good. You don't need to be authenticated to call this"}
	ctx.JSON(iris.StatusOK, response)
}

func SecuredPingHandler(ctx *iris.Context) {
	response := Response{"All good. You only get this message if you're authenticated"}
	ctx.JSON(iris.StatusOK, response)
}
```go