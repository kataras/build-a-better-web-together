# Web Development with Iris

Go is a great technology stack for building scalable, web-based, back-end systems for web 
applications. 

When you think about building web applications and web APIs, or simply building HTTP servers in Go, your mind goes to the standard net/http package. Then you have to deal with some common situations like the dynamic routing (a.k.a parameterized), security and authentication, real-time communication and many others which we will analyze later. 
Obviously the net/http package is not enough to build well-designed back-end systems for web, here is where **Iris web framework** comes to play.

## Table of Contents

- [Features](features.md)
- [Versioning](versioning.md)
- [Install](install.md)
- [Hi](hi.md)
- [TLS](tls.md)
- [Handlers](handlers.md)
 - [Using Handlers](using-handlers.md)
 - [Using HandlerFuncs](using-handlerfuncs.md)
 - [Using Annotated](using-annotated.md)
 - [Using native http.Handler](using-native-httphandler.md)
	   - [Using native http.Handler via iris.ToHandlerFunc()](using-native-httphandler-via-tohandlerfunc.md)
- [Middlewares](middlewares.md)
- [API](api.md)
- [Declaration](declaration.md)
- [Party](party.md)
- [Named Parameters](named-parameters.md)
- [Static files](static-files.md)
- [Send files](send-files.md)
- [Custom HTTP Errors](custom-http-errors.md)
- [Streaming](streaming.md)
- [Graceful](graceful.md)
- [Context](context.md)
- [Plugins](plugins.md)
- [Internationalization and Localization](internationalization-and-localization.md)
- [Examples](https://github.com/iris-contrib/examples)
