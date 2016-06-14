# Configuration

Configuration is a relative package `github.com/kataras/iris/config` 

>  No need to download it separately, it's downloaded automatically when you install Iris.

### Why?
I took this decision after a lot of thought and I ensure you that this is the best and easiest
architecture:

- change the configs without needing to re-write all of their fields.
	```go
	irisConfig := config.Iris{ DisablePathCorrection: true }
	api := iris.New(irisConfig)
	```
- **easy to remember**: `iris` type takes `config.Iris`, sessions takes `config.Sessions`, `iris.Config.Render` is of type `config.Render`, `iris.Config.Render.Template` is the type `config.Template`, `Logger` takes `config.Logger` and so on...

- **easy to search & find out what features exists and what you can change**: just navigate to the config folder and open the type you want to learn about, for example `/websocket`  package's configuration is in `/config/websocket.go`

- **All structs that receive configurations are already set to sane defaults** , so for casual use you can ignore them, but if you ever need to check them, you can find their default configs by this pattern: for example `config.Template` has `config.DefaultTemplate()`, `config.Rest` has `config.DefaultRest()`, `config.Typescript()` has `config.DefaultTypescript()`. Note however that `config.Iris` uses `config.Default()`. Even the plugins have their own default configs, to make it easier for you. 

- Enables you to do this **without setting up a config yourself**: `iris.Config.Render.Template.Engine = config.PongoEngine` or `iris.Config.Render.Template.Pongo.Extensions = []string{".xhtml", ".html"}`.

- **(Advanced usage) merge configs**: 

```go
//...
import "github.com/kataras/iris/config"
//...
templateFromRoutine1 := config.DefaultTemplate()
//....
templateFromOthers := config.Template{ Directory: "views"}

templateConfig := templateFromRoutine1.MergeSingle(templateFromOthers)

iris.Config.Render.Template = templateConfig

```


## [All Configs](https://github.com/kataras/iris/tree/master/config)
