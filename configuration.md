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

- **All structs which receives configuration are already default-setted** , so don't worry too much, but if you ever need them you can find their default configs by this pattern: for example `config.Template` has `config.DefaultTemplate()`, `config.Rest` has `config.DefaultRest()`, `config.Typescript()` has `config.DefaultTypescript()`, note that only `config.Iris` has `config.Default()`. Eeven the plugins have their default configs, to make it easier for you. 

- so you can do this **without pre-set a config by yourself**: `iris.Config().Render.Template.Engine = config.PongoEngine` or `iris.Config().Render.Template.Pongo.Extensions = []string{".xhtml", ".html"}`.

- **(Advanced usage) merge configs**: 

```go
//...
import "github.com/kataras/iris/config"
//...
templateFromRoutine1 := config.DefaultTemplate()
//....
templateFromOthers := config.Template{ Directory: "views"}

templateConfig := templateFromRoutine1.MergeSingle(templateFromOthers)

iris.Config().Render.Template = templateConfig

```

## List of the structures

```go
    // Iris configs for the station
	// All fields can be changed before server's listen except the PathCorrection field
	//
	// MaxRequestBodySize is the only options that can be changed after server listen -
	// using Config().MaxRequestBodySize = ...
	// Render's rest config can be changed after declaration but before server's listen -
	// using Config().Render.Rest...
	// Render's Template config can be changed after declaration but before server's listen -
	// using Config().Render.Template...
	// Sessions config can be changed after declaration but before server's listen -
	// using Config().Sessions...
	// and so on...
	Iris struct {
		// MaxRequestBodySize Maximum request body size.
		//
		// The server rejects requests with bodies exceeding this limit.
		//
		// By default request body size is unlimited.
		MaxRequestBodySize int
		// PathCorrection corrects and redirects the requested path to the registed path
		// for example, if /home/ path is requested but no handler for this Route found,
		// then the Router checks if /home handler exists, if yes,
		// (permant)redirects the client to the correct path /home
		//
		// Default is true
		PathCorrection bool

		// Log turn it to false if you want to disable logger,
		// Iris prints/logs ONLY errors, so be careful when you disable it
		Log bool

		// Profile set to true to enable web pprof (debug profiling)
		// Default is false, enabling makes available these 7 routes:
		// /debug/pprof/cmdline
		// /debug/pprof/profile
		// /debug/pprof/symbol
		// /debug/pprof/goroutine
		// /debug/pprof/heap
		// /debug/pprof/threadcreate
		// /debug/pprof/pprof/block
		Profile bool

		// ProfilePath change it if you want other url path than the default
		// Default is /debug/pprof , which means yourhost.com/debug/pprof
		ProfilePath string

		// Sessions the config for sessions
		// contains 3(three) properties
		// Provider: (look /sessions/providers)
		// Secret: cookie's name (string)
		// Life: cookie life (time.Duration)
		Sessions Sessions

		// Render contains the configs for template and rest configuration
		Render Render
	}

	// Render struct keeps organise all configuration about rendering, templates and rest currently.
	Render struct {
		// Template the configs for template
		Template Template
		// Rest configs for rendering.
		//
		// these options inside this config don't have any relation with the TemplateEngine
		// from github.com/kataras/iris/rest
		Rest Rest
	}
    
    
    
// Rest is a struct for specifying configuration options for the rest.Render object.
	Rest struct {
		// Appends the given character set to the Content-Type header. Default is "UTF-8".
		Charset string
		// Gzip enable it if you want to render with gzip compression. Default is false
		Gzip bool
		// Outputs human readable JSON.
		IndentJSON bool
		// Outputs human readable XML. Default is false.
		IndentXML bool
		// Prefixes the JSON output with the given bytes. Default is false.
		PrefixJSON []byte
		// Prefixes the XML output with the given bytes.
		PrefixXML []byte
		// Unescape HTML characters "&<>" to their original values. Default is false.
		UnEscapeHTML bool
		// Streams JSON responses instead of marshalling prior to sending. Default is false.
		StreamingJSON bool
		// Disables automatic rendering of http.StatusInternalServerError
        // when an error occurs. Default is false.
		DisableHTTPErrorRendering bool
	}

	EngineType uint8

	Template struct {
		// contains common configs for both standar & pongo
		Engine        EngineType
		Gzip          bool
		IsDevelopment bool
		Directory     string
		Extensions    []string
		ContentType   string
		Charset       string
		Asset         func(name string) ([]byte, error)
		AssetNames    func() []string
		Layout        string
		Standar       Standar // contains specific configs for standar html/template
		Pongo         Pongo   // contains specific configs for pongo2
	}

	Standar struct {
		RequirePartials bool
		// Delims
		Left  string
		Right string
		// Funcs for Standar
		Funcs []template.FuncMap
	}

	Pongo struct {
		// Filters for pongo2, map[name of the filter] the filter function
        // The filters are auto register
		Filters map[string]pongo2.FilterFunction
	}
    
    

```


