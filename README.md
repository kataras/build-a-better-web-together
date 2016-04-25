# Web Development with Iris

Go is a great technology stack for building scalable, web-based, back-end systems for web 
applications. 

When you think about building web applications and web APIs, or simply building HTTP servers in Go, your mind goes to the standard net/http package. Then you have to deal with some common situations like the dynamic routing (a.k.a parameterized), security and authentication, real-time communication and many others which we will analyze later. 
Obviously the net/http package is not enough to build well-designed back-end systems for web, here is where **Iris web framework** comes to play.



## Features
* **Typescript**: Auto-compile & Watch your client side code via the [typescript plugin](https://github.com/kataras/iris/tree/development/plugin/typescript)
* **Online IDE**: Edit & Compile your client side code when you are not home via the [editor plugin](https://github.com/kataras/iris/tree/development/plugin/editor)
* **Iris Online Control**: Web-based interface to control the basics functionalities of your server via the [iriscontrol plugin](https://github.com/kataras/iris/tree/development/plugin/iriscontrol). Note that Iris control is still young
* **Subdomains**: Easy way to express your api via custom and dynamic subdomains[*](https://github.com/iris-contrib/examples/blob/master/subdomains_simple)
* **Named Path Parameters**: Probably you already know what that means. If not, [It's easy to learn about](#named-parameters)
* **Custom HTTP Errors**: Define your own html templates or plain messages when http errors occurs[*](#custom-http-errors)
* **I18n**: [Internationalization](https://github.com/iris-contrib/examples/tree/master/middleware_internationalization_i18n)
* **Bindings**: Need a fast way to convert data from body or form into an object? Take a look [here](https://github.com/iris-contrib/examples/tree/master/bind_form_simple)
* **Streaming**: You have only one option when streaming comes in game[*](#streaming)
* **Middlewares**: Create and/or use global or per route middlewares with the Iris' simplicity[*](#middlewares)
* **Sessions**:  Sessions and secure cookies to provide a secure way to authenticate your clients/users [*](https://github.com/kataras/iris/tree/development/sessions)
* **Realtime**: Realtime is fun when you use websockets[*](https://github.com/kataras/iris/tree/development/websocket)
* **Context**: [Context](#context) is used for storing route params, storing handlers, sharing variables between middlewares, render rich content, send file and much more[*](#context)
* **Plugins**: You can build your own plugins to  inject the Iris framework[*](#plugins)
* **Full API**: All http methods are supported[*](#api)
* **Party**:  Group routes when sharing the same resources or middlewares. You can organise a party with domains too! [*](#party)
* **Transport Layer Security**: Provide privacy and data integrity between your server and the client[*](#tls)
* **Multi server instances**: Besides the fact that Iris has a default main server. You can declare as many as you need[*](#declaration)
* **Zero allocations**: Iris generates zero garbage

## Table of Contents
 
- [Install](#install)
- [Introduction](#introduction)
- [TLS](#tls)
- [Handlers](#handlers)
 - [Using Handlers](#using-handlers)
 - [Using HandlerFuncs](#using-handlerfuncs)
 - [Using Annotated](#using-annotated)
 - [Using native http.Handler](#using-native-httphandler)
    - [Using native http.Handler via iris.ToHandlerFunc()](#Using-native-http.Handler-via-ToHandlerFunc())
- [Middlewares](#middlewares)
- [API](#api)
- [Declaration & Options](#declaration)
- [Party](#party)
- [Named Parameters](#named-parameters)
- [Catch all and Static serving](#match-anything-and-the-static-serve-handler)
- [Custom HTTP Errors](#custom-http-errors)
- [Streaming](#streaming)
- [Graceful](#graceful)
- [Context](#context)
- [Plugins](#plugins)
- [Internationalization and Localization](https://github.com/iris-contrib/examples/tree/master/middleware_internationalization_i18n)
- [Examples](https://github.com/iris-contrib/examples)
- [Benchmarks](#benchmarks)
- [Third Party Middlewares](#third-party-middlewares)
- [Contributors](#contributors)
- [Community](#community)
- [Todo](#todo)
- [External source articles](#articles)
- [Versioning](#versioning)
- [License](#license)
