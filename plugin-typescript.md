# Typescript

[This is a plugin](https://github.com/kataras/iris/tree/development/plugin/typescript)

This is an Iris and typescript bridge plugin.

### What?

1. Search for typescript files (.ts)
2.    Search for typescript projects (.tsconfig)
3.    If 1 || 2 continue else stop
4.    Check if typescript is installed, if not then auto-install it (always inside npm global modules, -g)
5.    If typescript project then build the project using tsc -p $dir
6.    If typescript files and no project then build each typescript using tsc $filename
7.    Watch typescript files if any changes happens, then re-build (5|6)

 >Note: Ignore all typescript files & projects whose path has '/node_modules/'


### Options

 - **Bin**: string, the typescript installation path/bin/tsc or tsc.cmd, if empty then it will search to the global npm modules
 - **Dir**: string, Dir set the root, where to search for typescript files/project. Default "./" 
 - **Ignore**: string, comma separated ignore typescript files/project from these directories. Default "" (node_modules are always ignored) 
 - **Tsconfig**: &typescript.Tsconfig{}, here you can set all compilerOptions if no tsconfig.json exists inside the 'Dir' 
 - **Editor**: typescript.Editor(), if setted then alm-tools browser-based typescript IDE will be available. Defailt is nil
 
 >All these are optional


### How to use

```go
package main

import (
    "github.com/kataras/iris"
    "github.com/kataras/iris/plugin/typescript"
)

func main(){
    ts := typescript.Options {
        Dir: "./scripts/src",
        Tsconfig: &typescript.Tsconfig{Module: "commonjs", Target: "es5"}, 
    }
    // or typescript.DefaultTsconfig()

    iris.Plugin(typescript.New(ts)) //or with the default options just: typescript.New()

    iris.Get("/", func (ctx *iris.Context){})

    iris.Listen()
}
```

Enable [web browser editor](plugin-editor.md)

```go
ts := typescript.Options {
    //...
    Editor: typescript.Editor("username","passowrd")
    //...
}

```