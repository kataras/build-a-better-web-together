# Build a better web, together.


![cover](https://raw.githubusercontent.com/kataras/iris/gh-pages/assets/book/cover_1.png)

Go is a great technology stack for building scalable, web-based, back-end systems for web 
applications. 

When you think about building web applications and web APIs, or simply building HTTP servers in Go, your mind goes to the standard net/http package(?)
Then you have to deal with some common situations like the dynamic routing (a.k.a parameterized), security and authentication, real-time communication and many others that standard package doesn't provides. 

Obviously the net/http package is not enough to build well-designed back-end systems for web. But when you realize that, other thoughts are coming to your head:

- Ok the net/http package doesn't suits me, but they're so many frameworks, which I have to choose from?!
- Each one of them tells me that it's the best. I don't know what to do!

####The truth

I did a big research and benchmarks with 'wrk' and 'ab' in order to choose which framework suits me and my new project. The results, sadly, were really beaten me, disappointed me.

I was wondering if golang wasn't so fast on the web as I was reading... but, before let Golang and continue to develop with nodejs I told myself:

> '**Makis, don't lose your hope, give at least a chance to the Golang. Try to build something totally alone without being affected from the "slow" code you saw earlier, learn the secrets of this language and make *others* follow your steps!**'.



I'm not kidding, these are pretty much the words I told to myself that day [**13 March 2016**]. 

The same day, later the night, I was reading a book about Greek mythology, there I saw an ancient God's name, insipired immediately and give a name to this new web framework, which was started be written, to **Iris**.

**After two months**, I'm writing this intro. 

#### I am writing this book, I am coding with Golang [because Iris has succeed to be the fastest go web framework](https://github.com/kataras/iris#benchmarks)!

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
* [Render](render.md)
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

<form action="https://www.paypal.com/cgi-bin/webscr" method="post" target="_top">
<input type="hidden" name="cmd" value="_s-xclick">
<input type="hidden" name="encrypted" value="-----BEGIN PKCS7-----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-----END PKCS7-----
">
<input type="image" src="https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif" border="0" name="submit" alt="PayPal - The safer, easier way to pay online!">
<img alt="" border="0" src="https://www.paypalobjects.com/en_US/i/scr/pixel.gif" width="1" height="1">
</form>

