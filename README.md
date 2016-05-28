<a href ="https://github.com/kataras/iris"> <img src="http://iris-go.com/assets/book/cover_1.png" width="300" /> </a>


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
   * [Using native http.Handler](using-native-httphandler.md)
       * [Using native http.Handler via iris.ToHandlerFunc()](using-native-httphandler-via-tohandlerfunc.md)
* [Middlewares](middlewares.md)
* [API](api.md)
* [Declaration](declaration.md)
* [Configuration](configuration.md)
* [Party](party.md)
* [Subdomains](subdomains.md)
* [Named Parameters](named-parameters.md)
* [Static files](static-files.md)
* [Send files](send-files.md)
* [Render](render.md)
   * [REST](render_rest.md)
   * [Templates](render_templates.md)
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
* [Examples](https://github.com/iris-contrib/examples)


### Why

Go is a great technology stack for building scalable, web-based, back-end systems for web 
applications. 

When you think about building web applications and web APIs, or simply building HTTP servers in Go, your mind goes to the standard net/http package(?)
Then you have to deal with some common situations like the dynamic routing (a.k.a parameterized), security and authentication, real-time communication and many others that standard package doesn't provides. 

Obviously the net/http package is not enough to build well-designed back-end systems for web. But when you realize that, other thoughts are coming to your head:

- Ok the net/http package doesn't suits me, but they're so many frameworks, which I have to choose from?!
- Each one of them tells me that it's the best. I don't know what to do!

##### The truth

I did a big research and benchmarks with 'wrk' and 'ab' in order to choose which framework suits me and my new project. The results, sadly, were really beaten me, disappointed me.

I was wondering if golang wasn't so fast on the web as I was reading... but, before let Golang and continue to develop with nodejs I told myself:

> '**Makis, don't lose your hope, give at least a chance to the Golang. Try to build something totally alone without being affected from the "slow" code you saw earlier, learn the secrets of this language and make *others* follow your steps!**'.



I'm not kidding, these are pretty much the words I told to myself that day [**13 March 2016**]. 

The same day, later the night, I was reading a book about Greek mythology, there I saw an ancient God's name, inspired immediately and give a name to this new web framework, which was started be written, to **Iris**.

**After two months**, I'm writing this intro. 

 I'm still here [because Iris has succeed to be the fastest go web framework](https://github.com/kataras/iris#benchmarks)



