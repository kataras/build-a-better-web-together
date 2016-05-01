# Web Development with Iris

Go is a great technology stack for building scalable, web-based, back-end systems for web 
applications. 

When you think about building web applications and web APIs, or simply building HTTP servers in Go, your mind goes to the standard net/http package. Then you have to deal with some common situations like the dynamic routing (a.k.a parameterized), security and authentication, real-time communication and many others which we will analyze later. 
Obviously the net/http package is not enough to build well-designed back-end systems for web, here is where **Iris web framework** comes to play.

## Table of Contents

- [Features](#features)
- [Versioning](#versioning)
- [Install](#install)
- [Introduction](#introduction)
- [TLS](#tls)
- [Handlers](#handlers)
 - [Using Handlers](#using-handlers)
 - [Using HandlerFuncs](#using-handlerfuncs)
 - [Using Annotated](#using-annotated)
 - [Using native http.Handler](#using-native-httphandler)
	   - [Using native http.Handler via iris.ToHandlerFunc()](#Using-native-http.Handler-via-ToHandlerFunc()
- [Middlewares](#middlewares)
- [API](#api)
- [Declaration & Options](#declaration)
- [Party](#party)
- [Named Parameters](#named-parameters)
- [Static files](#static-files)
- [Send files](#send-files)
- [Custom HTTP Errors](#custom-http-errors)
- [Streaming](#streaming)
- [Graceful](#graceful)
- [Context](#context)
- [Plugins](#plugins)
- [Internationalization and Localization](https://github.com/iris-contrib/examples/tree/master/middleware_internationalization_i18n)
- [Examples](https://github.com/iris-contrib/examples)
