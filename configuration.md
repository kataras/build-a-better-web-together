# Configuration

Configuration owns the relative package `github.com/kataras/iris/config` 

>  No need to download it separately, it's being downloaded automatically when you installed Iris.

### Why?
I took this decision after a lot of thought and I ensure you that this is the best
architecture to easy:

- change the configs without need to re-write all of their fields.
	```go
	irisConfig := config.Iris { Profile: true, PathCorrection: false }
	api := iris.New(irisConfig)
	```
- **easy to remember**: `iris` type takes `config.Iris`, sessions takes `config.Sessions`, `iris.Config().Render` is the `config.Render`, `iris.Config().Render.Template` is the `config.Template`, `Logger` takes `config.Logger` and so on...

- **easy to search & find out what features are exists and what you can change**: just navigate to the config folder and open the type you want to learn about, for example `/iris.go` Iris' type configuration is on `/config/iris.go`

- **All structs which receives configuration are already default-setted** , so don't worry too much, but if you ever need them you can find their default configs by this pattern: for example `config.Template` has `config.DefaultTemplate()`, `config.Rest` has `config.DefaultRest()`, `config.Typescript()` has `config.DefaultTypescript()`, note that only `config.Iris` has `config.Default()`. I wrote that all structs even the plugins have their default configs now, to make it easier for you, so you can do this without set a config by yourself: `iris.Config().Render.Template.Engine = config.PongoEngine` or `iris.Config().Render.Template.Pongo.Extensions = []string{".xhtml", ".html"}`.


