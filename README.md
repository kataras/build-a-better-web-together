# Build a better web

![cover](https://raw.githubusercontent.com/kataras/iris/gh-pages/assets/book/cover_1.png)

Go is a great technology stack for building scalable, web-based, back-end systems for web 
applications. 

When you think about building web applications and web APIs, or simply building HTTP servers in Go, your mind goes to the standard net/http package(?)
Then you have to deal with some common situations like the dynamic routing (a.k.a parameterized), security and authentication, real-time communication and many others that standard package doesn't provides. 

Obviously the net/http package is not enough to build well-designed back-end systems for web. But when you realize that, other thoughts are coming to your head:

- Ok the net/http package doesn't suits me, but they're so many frameworks, which I have choose?!
- Each one of them tells me that it's the best. I don't know what to do!

####The truth

I did a big research and benchmarks in order to choose which framework suits me and my new project. The results were really hurt me, disappointed me.

I was wondering if golang wasn't so fast on the web as I was reading, but, before let Golang and continue to develop with nodejs I told myself:

> '**Makis, don't lose your hope, give at least a chance to the Golang. Try to build something totally alone without being affected from *others* code, learn the secrets of this language and make *others* to follow your steps!**'.



I'm not kidding, these are pretty much the words I told to myself that day [**13 March 2016**].

**After two months**, I'm writing this intro. 

#### I am writing this book, I am coding with Golang [because Iris has succeed](https://github.com/kataras/iris#benchmarks).

## Table of Contents

* [Introduction](README.md)
* [Features](features.md)
* [Versioning](versioning.md)
* [Install](install.md)
* [Hi](hi.md)
* [Transport Layer Security](tls.md)
* [Handlers](handlers.md)
   * [Using Handlers](using-handlers.md)
   * [Using HandlerFuncs](using-handlerfuncs.md)
   * [Using Annotated](using-annotated.md)
   * [Using native http.Handler](using-native-httphandler.md)
       * [Using native http.Handler via iris.ToHandlerFunc()](using-native-httphandler-via-tohandlerfunc.md)
* [Middlewares](middlewares.md)
* [API](api.md)
* [Declaration](declaration.md)
* [Party](party.md)
* [Subdomains](subdomains.md)
* [Named Parameters](named-parameters.md)
* [Static files](static-files.md)
* [Send files](send-files.md)
* [Gzip](gzip.md)
* [Streaming](streaming.md)
* [Cookies](cookies.md)
* [Flash messages](flashmessages.md)
* [Body binder](request-body-bind.md)
* [Custom HTTP Errors](custom-http-errors.md)
* [Context](context.md)
* [Logger](logger.md)
* [HTTP access control](middleware-cors.md)
* [Secure](middleware-secure.md)
* [Sessions](package-sessions.md)
* [Websockets](package-websocket.md)
* [Graceful](package-graceful.md)
* [Recovery](middleware-recovery.md)
* [Plugins](plugins.md)
* [Internationalization and Localization](middleware-internationalization-and-localization.md)
* [Easy Typescript](plugin-typescript.md)
* [Browser based Editor](plugin-editor.md)
* [Routes info](plugin-routesinfo.md)
* [Control panel](plugin-iriscontrol.md)
* [Examples](https:/github.com/iris-contrib/examples)

